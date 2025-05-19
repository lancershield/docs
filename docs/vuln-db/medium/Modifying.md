# Modifying Storage Array by Value Causes Disconnected State Updates

```YAML
id: TBA
title: Modifying Storage Array by Value Causes Disconnected State Updates
severity: M
category: storage
language: solidity
blockchain: [ethereum]
impact: State desynchronization, logic failure, or ineffective updates
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-710
swc: SWC-109
```

## 📝 Description

- In Solidity, accessing a storage array element by value rather than by reference leads to silent bugs where updates do not persist in storage. When developers write:
- The modification applies only to a temporary in-memory copy—not to the storage-backed array. 
- This leads to logic errors, where updates appear successful during execution but vanish once the transaction completes. 
- The bug is common when modifying structs within storage arrays.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract UserRegistry {
    struct User {
        uint256 balance;
        bool active;
    }

    User[] public users;

    function updateFirstUser(uint256 newBalance) external {
        User memory user = users[0]; // ❌ Copy by value
        user.balance = newBalance;   // Change lost after function ends
    }

    function addUser() external {
        users.push(User(0, true));
    }
}
```

## 🧪 Exploit Scenario

1. Contract stores a list of User structs in a dynamic array.
2. A developer calls users[0] into memory and modifies its fields.
3. The state change is expected to persist but does not.
4. Other contract logic assumes the user was updated, resulting in incorrect access control, incorrect balances, or broken reward logic.

**Assumptions:**

- Developers mistakenly believe user = users[0] modifies storage.
- No state update verification or post-condition checks exist.
- Function logic proceeds under false assumptions of success.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeUserRegistry {
    struct User {
        uint256 balance;
        bool active;
    }

    User[] public users;

    function updateFirstUser(uint256 newBalance) external {
        User storage user = users[0]; // ✅ Reference storage directly
        user.balance = newBalance;
    }

    function addUser() external {
        users.push(User(0, true));
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always use the storage keyword when modifying array elements.
- Be cautious when assigning structs or arrays from storage.

### Additional Safeguards

- Use comments or naming conventions to distinguish memory vs storage.
- Use internal setter functions to encapsulate safe mutation logic.
- Add events or post-assertions to confirm updates.

### Detection Methods

- Check for struct assignments like T memory var = storageArray[i].
- Detect updates to memory variables that are never written back.
- Tools: Slither (uninitialized-storage, useless-assignment), MythX, manual audit

## 🕰️ Historical Exploits

- **Name:** Storage Array Mismanagement in Early DeFi Contracts
- **Date:** 2020 
- **Loss:** Logic errors leading to incorrect state updates 
- **Post-mortem:** [Link to post-mortem](https://ethereum.stackexchange.com/questions/124325/why-does-this-function-not-update-my-storage-array) 
  
## 📚 Further Reading

- [SWC-109: Uninitialized Storage Pointer](https://swcregistry.io/docs/SWC-109/) 
- [Solidity Docs – Storage and Memory](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#storage-memory-and-the-stack) 
- [Solidity Documentation: Data Location](https://docs.soliditylang.org/en/latest/types.html#data-location) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Modifying Storage Array by Value Causes Disconnected State Updates
severity: M
score:
impact: 3         
exploitability: 2 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 3.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Leads to state inconsistencies or logic failure; may void critical updates.
- **Exploitability**: Not directly user-exploitable, but easy to introduce via dev error.
- **Reachability**: Often occurs in staking, voting, and reward contracts using arrays.
- **Complexity**: Low; mistake arises from misunderstanding memory vs storage behavior.
- **Detectability**: Easy to overlook; updates seem correct unless verified with state checks or events.