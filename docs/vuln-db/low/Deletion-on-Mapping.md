# Deletion on Mapping

```YAML
id: TBA
title: Deletion on Mapping
baseSeverity: L
category: storage
language: solidity
blockchain: [ethereum]
impact: Inconsistent state, gas griefing, or minor DoS
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.5.0", "<latest"]
cwe: CWE-704
swc: SWC-135
```
## 📝 Description

- In Solidity, deleting a mapping entry that contains a struct via delete mapping[key] resets all fields of the struct to their default values (e.g., 0, false, address(0)). 
- However, this may not fully remove logical relationships or presence flags stored outside the struct or within it. 
- If other parts of the contract depend on the struct’s non-zero fields to check validity, deletion can create ambiguous or exploitable states.
- Common issues include:
- Incorrectly assuming that delete removes the mapping key itself (it doesn't—it resets the value to the default struct).
- Reusing struct fields as logic flags (e.g., if (user.exists) or if (user.balance > 0)).
- Leaving residual entries in iteration logic or enumerable mappings.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract StructDelete {
    struct User {
        uint256 balance;
        bool isActive;
    }

    mapping(address => User) public users;

    function removeUser(address user) external {
        delete users[user]; // ❌ Sets values to 0/false, but key still "exists"
    }

    function isRegistered(address user) external view returns (bool) {
        return users[user].isActive; // Will return false even for old users
    }
}
```

## 🧪 Exploit Scenario

1. Contract tracks user registration with a struct: {balance, isActive}.
2. delete users[alice] is called after she withdraws funds.
3. Her entry remains in users, but all fields are zeroed.
4. A re-registration check like require(!users[alice].isActive) passes, but other logic (e.g., referral counters, index mapping) still refers to her address.
5. This causes accounting mismatches, unintended rewards, or logic corruption.

**Assumptions:**

- Mapping keys are reused or iterated externally.
- Struct field values are used as logical flags.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeStructDelete {
    struct User {
        uint256 balance;
        bool isActive;
    }

    mapping(address => User) public users;
    mapping(address => bool) public exists;

    function addUser(address user) external {
        require(!exists[user], "Already registered");
        users[user] = User(0, true);
        exists[user] = true;
    }

    function removeUser(address user) external {
        delete users[user];
        exists[user] = false;
    }

    function isRegistered(address user) external view returns (bool) {
        return exists[user]; // ✅ Clear presence check
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: L
  reasoning: "Logical confusion, but unlikely to cause major loss."
- context: "Governance or staking-based system"
  severity: M
  reasoning: "Can be abused to reclaim rewards or bypass eligibility checks."
- context: "Private use contract with full control"
  severity: I
  reasoning: "Has no impact under full admin control and no logic tied to deletion."
```

## 🛡️ Prevention

### Primary Defenses

- Never assume delete mapping[key] removes the key completely—it only resets the value.
- Always track existence explicitly if needed for logic control or external iteration.

### Additional Safeguards

- Use a parallel presence-tracking mapping (e.g., exists[address]).
- For enumerable mappings, explicitly manage entry lists and indexes.

### Detection Methods

- Search for use of delete mapping[key] in structs without presence flags.
- Check for functions relying on struct field values to infer mapping key existence.
- Tools: Slither (struct-default-check, logic-deletion), manual logic review

## 🕰️ Historical Exploits

- **Name:** Akropolis Struct Deletion Bug 
- **Date:** 2020 
- **Loss:** ~$2M 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/akropolis-rekt/) 

## 📚 Further Reading

- [SWC-135: Incorrect Authorization Logic](https://swcregistry.io/docs/SWC-135/) 
- [Solidity Docs – delete on Mappings](https://docs.soliditylang.org/en/latest/types.html#delete) 
- [Trail of Bits: Struct Deletion Pitfalls](https://github.com/crytic/slither/wiki/Detector-Documentation#delete-on-structs) 
- [OpenZeppelin: Enumerable Mapping Caveats](https://docs.openzeppelin.com/contracts/4.x/api/utils#EnumerableMap) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Deletion on Mapping 
severity: L
score:
impact: 3         
exploitability: 3 
reachability: 3   
complexity: 2     
detectability: 3 
finalScore: 3.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Logic can break due to relying on default values to imply non-existence.
- **Exploitability**: Attackers can re-register, claim rewards, or bypass eligibility checks.
- **Reachability**: Common in user mappings, NFT registries, or game state tracking.
- **Complexity**: Easy to introduce; based on misunderstanding of delete.
- **Detectability**: Not obvious unless mapping usage and logic are closely reviewed.
