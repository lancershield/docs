# Uninitialized State Variables Lead to Undefined Behavior and Access Control Risks

```YAML
id: TBA
title: Uninitialized State Variables Lead to Undefined Behavior and Access Control Risks
severity: H
category: initialization
language: solidity
blockchain: [ethereum]
impact: Incorrect logic execution, unauthorized access, or asset loss
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-457
swc: SWC-118
```

## 📝 Description

- In Solidity, failing to explicitly initialize state variables can lead to critical security issues. 
- While Solidity provides default values for uninitialized variables (e.g., 0 for uint, address(0) for address), relying on these defaults unintentionally can break access control, logic gates, or permission assumptions.
- This vulnerability commonly appears in:
- Ownership contracts where owner is never set.
- Role-based systems where roles remain empty.
- Proxies and upgradable contracts where initialization logic is separated from constructors but not invoked.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Vault {
    address public owner; // ❌ Not initialized

    function withdraw() external {
        require(msg.sender == owner, "Not the owner");
        // withdraw logic...
    }
}
```

## 🧪 Exploit Scenario

1. A developer deploys a contract expecting themselves to be the owner.
2. They forget to set the owner in the constructor or initialization logic.
3. Attacker later finds a function that sets owner without restriction or calls initialize() on a proxy-based contract.
4. Attacker becomes the owner and drains funds using admin functions like withdraw() or upgrade().

**Assumptions:**

- Developers assume the variable will be initialized automatically or externally.
- There are no access guards to prevent late initialization by unauthorized actors.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SecureVault {
    address public owner;

    constructor() {
        owner = msg.sender; // ✅ Properly initialized
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }

    function withdraw() external onlyOwner {
        // withdraw logic...
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always initialize important state (e.g., owner, admin, oracle) during deployment or upgrade.
- Use constructors or initializer functions guarded by initializer modifiers.
- Use OpenZeppelin’s Initializable and Ownable utilities for upgradable contracts.

### Additional Safeguards

- Include constructor or initializer logic in deployment checklists.
- Use modifiers like onlyInitializing to prevent double or late initialization.

### Detection Methods

- Scan for critical state variables (e.g., owner, admin) that are declared but not assigned.
- Analyze upgradeable contracts for missing initializer invocations.
- Tools: Slither (uninitialized-state), Hardhat plugin, manual audit

## 🕰️ Historical Exploits

- **Name:** Akropolis Savings Pool Exploit 
- **Date:** 2020 
- **Loss:** ~$2M 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/akropolis-rekt/) 
  
## 📚 Further Reading

- [SWC-118: Uninitialized State Variables](https://swcregistry.io/docs/SWC-118/) 
- [OpenZeppelin: Initializable Contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializing-the-contract) 
- [Solidity Docs – Default Values of Variables](https://docs.soliditylang.org/en/latest/types.html#default-value) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Uninitialized State Variables Lead to Undefined Behavior and Access Control Risks
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 3   
complexity: 2     
detectability: 3  
finalScore: 3.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Without proper initialization, any admin function can become publicly accessible or completely unusable.
- **Exploitability**: Moderate—depends on deployment conditions or missing guards.
- **Reachability**: High for upgradable contracts, governance roles, and DAO vaults.
- **Complexity**: Very low—often a basic developer oversight.
- **Detectability**: Easy to miss in large codebases or clones; Slither can help catch it.







