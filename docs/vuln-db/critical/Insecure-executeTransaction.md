# Insecure executeTransaction

```YAML
id: TBA
title: Insecure executeTransaction
severity: C
category: access-control
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Arbitrary function execution and asset theft
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-306: Missing Authentication for Critical Function
swc: SWC-105: Unprotected Function
```

## 📝 Description

- The executeTransaction() function is typically used to perform arbitrary external calls—common in Gnosis-style multisigs, DAOs, or admin upgrade logic. 
- If this function is not protected by strict access control (e.g., onlyOwner, onlyGovernance), any user can execute arbitrary transactions from the contract’s balance or privileges.
- This leads to:
- Complete takeover of contract logic
- Transfer of funds or NFTs to attacker
- Governance bypass or upgrade hijack

## 🚨 Vulnerable Code

```solidity

function executeTransaction(address target, uint256 value, bytes calldata data) external {
    (bool success, ) = target.call{value: value}(data); // ❌ no access control
    require(success, "Tx failed");
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. An attacker finds a protocol with an unprotected executeTransaction() function.
2. This causes the contract to transfer tokens it holds to the attacker's address.
3. Alternatively, the attacker could upgrade proxy logic by calling upgradeTo() or take admin control of other modules.

**Assumptions:**

- The function is externally callable by any address (external or public).
- There is no access control modifier (e.g., onlyOwner, onlyGovernance).
- The contract holds assets, interacts with privileged logic, or manages critical infrastructure (e.g., proxy admin, governance router).
- No checks exist to restrict the target address or the calldata payload.

## ✅ Fixed Code

```solidity

function executeTransaction(address target, uint256 value, bytes calldata data) external onlyOwner {
    (bool success, ) = target.call{value: value}(data);
    require(success, "Tx failed");
}
```

## 🛡️ Prevention

### Primary Defenses

- Apply onlyOwner, onlyGovernance, or onlyMultisig modifiers to all arbitrary call functions.
- Use OpenZeppelin's AccessControl or Ownable to manage privileged access.
- Consider adding proposal queuing via TimelockController.

### Additional Safeguards

- Whitelist target contracts or method signatures.
- Log every execution with transparent ExecuteTransaction events.
- Require multi-sig or DAO approval before sensitive actions.

### Detection Methods

- Slither: unprotected-function, dangerous-low-level-call
- Manual review of functions using call, delegatecall, or executeTransaction
- Foundry fuzzing and symbolic execution tools (MythX, Manticore)

## 🕰️ Historical Exploits

- **Name:** xToken Arbitrary Execution Bug 
- **Date:** 2021-05 
- **Loss:** ~$25,000,000 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/xtoken-rekt/) 
  
## 📚 Further Reading

- [OpenZeppelin – AccessControl](https://docs.openzeppelin.com/contracts/4.x/access-control) 
- [Rekt – bZx Governance Hack](https://rekt.news/bzx-rekt/) 
- [Slither – Dangerous Calls](https://github.com/crytic/slither) 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Insecure executeTransaction Function
severity: C
score:
impact: 5      
exploitability: 4 
reachability: 4  
complexity: 2    
detectability: 4 
finalScore: 4.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Enables full fund drain, logic hijack, or proxy upgrades.
- **Exploitability**: Can be exploited via any unprivileged EOA or contract if unprotected.
- **Reachability**: Found in many governance or dev tooling contracts.
- **Complexity**: Low to moderate—any attacker can craft calldata.
- **Detectability**: Highly visible in code reviews and static analysis.


