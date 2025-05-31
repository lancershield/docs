# Upgrade Bricking

```YAML
id: LS10H
title: Upgrade Bricking 
baseSeverity: H
category: upgradeability
language: solidity
blockchain: [ethereum]
impact: Permanent loss of upgradeability or contract usability
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.6.0", "<0.8.21"]
cwe: CWE-665
swc: SWC-112
```

## 📝 Description

- Upgrade bricking occurs when a smart contract proxy is upgraded to a new implementation that:
- Contains a constructor instead of initializer,
- Lacks proper storage compatibility with the proxy,
- Misses essential initializer logic (e.g., `initialize()` not called),
- contains self-destructive logic or critical bugs.
- This renders the proxy  permanently unusable, locking funds or breaking functionality. In upgradeable systems (e.g., UUPS, Transparent), implementation logic must be meticulously audited and initialized before being assigned.

## 🚨 Vulnerable Code

```solidity
// Proxy contract (e.g., UUPS)
contract UpgradeableProxy {
    address public implementation;

    fallback() external payable {
        address impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }

    function upgradeTo(address newImpl) external {
        implementation = newImpl;
    }
}

// ❌ New logic has a constructor — runs only once, not via delegatecall
contract BrokenLogic {
    address public owner;

    constructor() {
        owner = msg.sender; // will not be set when used via delegatecall
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Proxy contract is upgraded to BrokenLogic.
2. Since constructor() is not run in delegatecall, owner remains uninitialized.
3. No initialize() exists, or the upgrade flow forgets to call it.
4. The new logic fails all permission checks or even reverts all calls.
5. Contract becomes permanently unusable unless a second upgrade fixes it — if that's even possible.

**Assumptions:**

- Upgraded implementation lacks proper initializer or breaks storage layout.
- Proxy upgrade function allows invalid contracts to be set without checks.

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract SafeLogic is Initializable {
    address public owner;

    function initialize(address _owner) public initializer {
        owner = _owner;
    }
}
```
## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: H
  reasoning: "Permanent loss of upgradeability or usability is severe, even without fund loss."
- context: "Protocol with Time-Locked Governance Upgrades"
  severity: M
  reasoning: "Time delay and peer review reduce the risk, but not impact."
- context: "Manually Upgraded Private App"
  severity: L
  reasoning: "Errors likely to be detected during direct testing before deployment."
```

## 🛡️ Prevention

### Primary Defenses

- Always use initializer()-based logic (not constructors) in upgradeable contracts.
- Enforce storage layout compatibility across all implementation versions.
- Use OpenZeppelin Upgrades Plugins for validated upgrade safety and initializer invocation.

### Additional Safeguards

- Write upgrade test scripts that simulate real proxy deployment + upgrade.
- Use validateUpgrade() or proxiableUUID() in UUPS pattern to reject incompatible implementations.
- Implement upgrade timelocks or governance approval flows to prevent rushed or malicious upgrades.

### Detection Methods

- Slither: constructor-in-upgradeable, storage-incompatibility, upgrade-to-broken rules.
- Manual inspection of initializer logic and storage layout diffs.
- Use openzeppelin-upgrades hardhat plugin to simulate upgrade safety checks.

## 🕰️ Historical Exploits

- **Name:** Parity Multisig Wallet Freeze 
- **Date:** 2017-11-06 
- **Loss:** Approximately 513,774 ETH 
- **Post-mortem:** [Link to post-mortem](https://codeofcode.org/lessons/case-studies-of-real-world-smart-contract-vulnerabilities-and-exploits/)

## 📚 Further Reading

- [SWC-112: Delegatecall to Untrusted Callee](https://swcregistry.io/docs/SWC-112) 
- [OpenZeppelin – Upgrade Patterns](https://docs.openzeppelin.com/upgrades-plugins/1.x/) 
- [EIP-1822: UUPS Proxy Standard](https://eips.ethereum.org/EIPS/eip-1822)
- [Solidity Docs – Initializers](https://docs.soliditylang.org/en/latest/contracts.html#constructors) 
  
---

## ✅ Vulnerability Report

```markdown
id: LS10H
title: Upgrade Bricking  
severity: H
score:
impact: 5         
exploitability: 3 
reachability: 4  
complexity: 3     
detectability: 4  
finalScore: 4.2
```

---

## 📄 Justifications & Analysis

- **Impact**: Breaks proxy entirely — functions revert or become inaccessible.
- **Exploitability**: High-risk if upgrades aren’t validated or tested.
- **Reachability**: Present in all upgradable contracts (UUPS, Transparent).
- **Complexity**: Medium — misuse or omission of initialize() or storage layout.
- **Detectability**: Easily flagged with OpenZeppelin's upgrade safety tooling.