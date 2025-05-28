# Uninitialized Storage Pointers

```YAML
id: TBA
title: Uninitialized Storage Pointers 
baseSeverity: H
category: storage-pointer
language: solidity
blockchain: [ethereum]
impact: Overwriting unrelated contract storage variables
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-824
swc: SWC-109
```

## 📝 Description

- Uninitialized storage pointers in Solidity can cause serious and unpredictable behavior.
- When a storage reference (e.g., `Struct s;`) is declared without the `memory` or `storage` keyword inside a function, Solidity defaults to storage slot `0`, which may overwrite important variables such as `owner`, `balances`, or mappings. 
- This leads to **unintended storage collisions** and potential total compromise of the contract’s logic or access control.

## 🚨 Vulnerable Code

```solidity
struct User {
    uint256 balance;
    bool isWhitelisted;
}

contract UninitializedPointer {
    mapping(address => User) public users;
    address public owner;

    function initUser() public {
        User user; // ❌ uninitialized storage pointer
        user.balance = 100;
        user.isWhitelisted = true;
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Function initUser() is called.
2. The uninitialized User user; pointer defaults to storage slot 0.
3. This overwrites owner and corrupts mappings like users[msg.sender].
4. Ownership is lost and mappings may become inaccessible or invalid.

**Assumptions:**

- Storage pointer declared without memory or storage.
- Contract has critical variables stored in early slots.

## ✅ Fixed Code

```solidity

function initUser() public {
    User memory user; // ✅ proper memory allocation
    user.balance = 100;
    user.isWhitelisted = true;

    users[msg.sender] = user; // save to actual storage mapping
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Allows overwriting of critical storage slots leading to privilege or logic corruption."
- context: "Contracts using proxy patterns"
  severity: C
  reasoning: "Storage slot overwrites can break upgradeable logic or admin controls."
- context: "Contracts with tight manual storage layout"
  severity: M
  reasoning: "Less likely if design ensures sensitive slots are isolated."
```

## 🛡️ Prevention

### Primary Defenses

- Always declare whether struct or array variables in functions are memory or storage.
- Avoid using Struct s; inside functions without explicit location.

### Additional Safeguards

- Use linters or Slither to detect uninitialized storage.
- Write tests to check that storage layout remains intact.

### Detection Methods

- Slither: uninitialized-storage and storage-pointer detectors.
- Mythril symbolic analysis can identify overwritten storage slots.
- Manual audit of any struct or array declarations in internal logic.

## 🕰️ Historical Exploits

 - **Name:** Balsn CTF 2019 – Bank Challenge 
 - **Date:** 2019-01-16 
 - **Loss:** Exploitation of contract logic via uninitialized storage pointer 
 - **Post-mortem:** [Link to post-mortem](https://x9453.github.io/2020/01/16/Balsn-CTF-2019-Bank/) 
 - **Name:** Uninitialized Storage Pointer Exploit Example 
 - **Date:** 2020-12-31 
 - **Loss:** Potential overwriting of critical contract storage 
 - **Post-mortem:** [Link to post-mortem](https://immunebytes.com/blog/a-complete-breakdown-of-uninitialized-storage-parameters/) 
  
## 📚 Further Reading

- [SWC-109: Uninitialized Storage Pointer](https://swcregistry.io/docs/SWC-109) 
- [Solidity Docs – Data Location](https://docs.soliditylang.org/en/latest/types.html#data-location) 
- [Sigma Prime – Solidity Security Blog](https://blog.sigmaprime.io/solidity-security.html) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Uninitialized Storage Pointers 
severity: H
score:
impact: 5         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 4.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Corrupts important variables like ownership or balances.
- **Exploitability**: Triggered by any external caller if the function is public.
- **Reachability**: Common in user setup functions or admin initializers.
- **Complexity**: Simple call; attacker does not need a contract.
- **Detectability**: Detected by tools like Slither; human audits often overlook it unless explicitly tested.
