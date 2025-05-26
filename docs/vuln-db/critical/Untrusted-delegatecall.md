# Untrusted delegatecall

```YAML
id: TBA
title: Untrusted delegatecall 
baseSeverity: C
category: delegatecall
language: solidity
blockchain: [ethereum]
impact: Full control of contract state and logic execution
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-829
swc: SWC-112
```

## 📝 Description

- The delegatecall opcode in Solidity allows a contract to execute code from another contract in the context of the caller's storage, msg.sender, and msg.value. 
- When used improperly—especially with untrusted or user-supplied addresses—delegatecall can allow arbitrary code execution, leading to:
- Storage hijacking and overwriting critical variables (e.g., owner, balances)
- Bypass of access control and logic constraints
- Complete takeover of contract behavior
- Because the callee runs in the storage of the caller, even a small external library or contract can completely corrupt the protocol if not tightly controlled.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Proxy {
    address public implementation;

    function upgrade(address _impl) external {
        implementation = _impl;
    }

    function execute(bytes calldata data) external payable {
        (bool success, ) = implementation.delegatecall(data); // ❌ untrusted delegatecall
        require(success, "Delegatecall failed");
    }
}
```

## 🧪 Exploit Scenario

1. A user sets implementation = attackerContract.
2. The attacker crafts a function in attackerContract to overwrite owner or call selfdestruct.
3. They call execute() with calldata for that function.
4. The proxy delegatecalls into the malicious code, executing it in the context of the proxy.
5. Ownership is stolen, or funds are drained.

**Assumptions:**

- delegatecall target address is untrusted or user-controlled.
- Storage layout is not safeguarded via versioning or access control.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SecureProxy {
    address public implementation;
    address public admin;

    modifier onlyAdmin() {
        require(msg.sender == admin, "Not authorized");
        _;
    }

    constructor(address _impl) {
        implementation = _impl;
        admin = msg.sender;
    }

    function upgrade(address _impl) external onlyAdmin {
        require(_impl != address(0), "Invalid impl");
        implementation = _impl;
    }

    fallback() external payable {
        address impl = implementation;
        require(impl != address(0), "No implementation");

        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())

            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Public delegatecall with user-supplied target"
  severity: C
  reasoning: "Complete takeover of caller's contract possible via arbitrary storage overwrite."
- context: "delegatecall to immutable address set at deployment"
  severity: M
  reasoning: "Risk mitigated by immutability and prior audit of logic."
- context: "Proxy logic with ownership and upgrade restrictions"
  severity: L
  reasoning: "Control surfaces are tightly gated; exploit unlikely unless upgrade key is compromised."
```

## 🛡️ Prevention

### Primary Defenses

- Never delegatecall into untrusted or user-supplied addresses.
- Lock upgrades behind onlyOwner or DAO-controlled permissions.
- Use upgradable proxy patterns with well-tested libraries like OpenZeppelin.

### Additional Safeguards

- Validate bytecode or interface of implementation before calling.
- Isolate upgrade logic into governance-controlled modules.
- Maintain exact storage layout to prevent clobbering.

### Detection Methods

- Scan for delegatecall() usage with dynamic or unverified targets.
- Audit access control on upgrade functions or dynamic delegates.
- Tools: Slither (untrusted-delegatecall), MythX, Foundry Fuzzing

## 🕰️ Historical Exploits

- **Name:** Parity Multisig Wallet Hack #2 
- **Date:** 2017 
- **Loss:** ~$280M 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert.html) 

📚 Further Reading

- [SWC-112: Delegatecall to Untrusted Contract](https://swcregistry.io/docs/SWC-112/) 
- [EIP-1967: Standard Proxy Storage Slots](https://eips.ethereum.org/EIPS/eip-1967) 
- [OpenZeppelin Upgradeable Contracts Guide](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies)
- [Slither – Untrusted Delegatecall Detector](https://github.com/crytic/slither/wiki/Detector-Documentation#untrusted-delegatecall)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Untrusted delegatecall 
severity: C
score:
impact: 5         
exploitability: 4 
reachability: 4  
complexity: 3  
detectability: 4  
finalScore: 4.4
```

---

## 📄 Justifications & Analysis

- **Impact**: Enables complete compromise of the contract, including asset theft.
- **Exploitability**: Trivial once attacker can set implementation address.
- **Reachability**: Widespread in proxy and modular design patterns.
- **Complexity**: Requires knowledge of delegatecall mechanics and slot alignment.
- **Detectability**: Easy to catch via Slither or manual review of proxy logic.