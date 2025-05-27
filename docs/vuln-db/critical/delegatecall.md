# Untrusted delegatecall Usage

```YAML
id: TBA
title: Untrusted delegatecall Usage
baseSeverity: C
category: proxy-logic
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Storage corruption, ownership hijack, or full contract compromise
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-829
swc: SWC-112
```

## 📝 Description

- Delegatecall is a low-level Solidity function that allows one contract to execute code from another in the context of the calling contract's storage. 
- This is foundational for proxy and upgradeable patterns. However, if misused or pointed at untrusted logic, delegatecall allows the callee to:
- Read and write arbitrary storage slots
- Change ownership or critical roles
- Re-enter and hijack execution flows
- Destroy the caller's logic via storage corruption
- When delegatecall is used with an unvalidated or user-supplied address, the entire contract becomes susceptible to takeover.

## 🚨 Vulnerable Code

```solidity

contract UnsafeProxy {
    address public implementation;

    function upgradeTo(address newImpl) external {
        implementation = newImpl;
    }

    fallback() external payable {
        address _impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), _impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A protocol uses an upgradeable proxy that allows upgradeTo() to be called by a governance module or admin.
2. The access control is mistakenly removed or compromised.
3. Attacker calls upgradeTo(attacker_contract).
4. Attacker becomes owner and drains funds or destroys logic.

**Assumptions:**

- delegatecall target address is user-controlled or upgradeable without checks
- Callee contract is untrusted or malicious
- Storage layout between proxy and logic contract is misaligned or exploitable

## ✅ Fixed Code

```solidity

contract SecureProxy is Ownable {
    address public immutable trustedImplementation;

    constructor(address _impl) {
        require(_impl != address(0), "Invalid");
        trustedImplementation = _impl;
    }

    fallback() external payable {
        address _impl = trustedImplementation;
        require(_impl != address(0), "Impl not set");
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), _impl, 0, calldatasize(), 0, 0)
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
- context: "Delegatecall to untrusted or user-controlled contract"
  severity: C
  reasoning: "Full contract takeover and fund loss possible"
- context: "Trusted upgrade path with access control but no layout checks"
  severity: H
  reasoning: "Partial corruption or undefined behavior may still occur"
- context: "Immutable implementation or strict upgrade gating"
  severity: L
  reasoning: "Delegatecall is safe if locked and aligned"
```

## 🛡️ Prevention

### Primary Defenses

- Always restrict upgrade functions with onlyOwner or onlyGovernance
- Verify that implementation contracts are audited and storage-compatible
- Use delegatecall only with static and verified addresses, not user inputs

### Additional Safeguards

- Use ERC1967, UUPS, or Transparent Proxy standards
- Implement rollback testing and slot collision detection
- Maintain an immutable upgrade registry for audit trails

### Detection Methods

- Slither: dangerous-delegatecall, unchecked-delegatecall
- Manual review of fallback functions and upgrade paths
- Symbolic testing for slot overwrites or privilege escalations

## 🕰️ Historical Exploits

- **Name:** Parity Multisig Wallet Library SelfDestruct
- **Date:** 2017 
- **Loss:** ~$280M frozen 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert.html) 
  
## 📚 Further Reading

- [SWC-112: Delegatecall to Untrusted Contract](https://swcregistry.io/docs/SWC-112)
- [CWE-829: Inclusion of Untrusted Code](https://cwe.mitre.org/data/definitions/829.html)
- [Solidity Docs – delegatecall](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#delegatecall-callcode-and-libraries)
- [OpenZeppelin Upgrades Plugins](https://docs.openzeppelin.com/upgrades-plugins/1.x/) 

---

✅ Vulnerability Report 

```markdown
id: TBA
title: Untrusted delegatecall Usage
severity: C
score:
impact: 5
exploitability: 4
reachability: 5
complexity: 2
detectability: 3
finalScore: 4.35
```

## 📄 Justifications & Analysis

- **Impact**: Grants attackers the ability to execute arbitrary code in the context of the calling contract, leading to fund theft, storage corruption, or total control
- **Exploitability**: Easily exploited if delegatecall target is user-controlled or unverified
- **Reachability**: High if upgrade path or fallback is exposed
- **Complexity**: Medium—attacker needs to craft compatible storage layout or malicious logic
- **Detectability**: Detectable with static tools or manual review, but commonly missed if deep in proxy internals