# Storage ABIEncoderV2 Array Corruption

```YAML
id: TBA
title: Storage ABIEncoderV2 Array Corruption
severity: H
category: storage
language: solidity
blockchain: [ethereum]
impact: Storage corruption, unexpected overwrites, or broken decoding
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.24", "<0.8.4"]
cwe: CWE-665
swc: SWC-124
```

## 📝 Description

- Storage ABIEncoderV2 Array Corruption is a vulnerability that arises when using Solidity’s ABIEncoderV2 feature (enabled via pragma experimental ABIEncoderV2) with dynamic arrays in storage prior to version 0.8.4.
- Improper encoding and decoding of nested arrays or structs can lead to data corruption, silent overwrites, or unexpected behavior when reading from or writing to storage.
- In affected versions, encoding a storage variable and decoding it back—especially involving dynamic arrays of structs—can write to overlapping storage slots or misinterpret offsets, breaking expected logic and leading to application failure or even fund loss.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.7.6;
pragma experimental ABIEncoderV2;

contract StorageCorruption {
    struct User {
        uint256 balance;
        string name;
    }

    User[] public users;

    function updateUser() public {
        users.push(User(100, "alice"));
        bytes memory data = abi.encode(users);
        User[] memory decoded = abi.decode(data, (User[]));
        // Decoding back to storage can cause issues if reassigned
        users = decoded; // ⚠️ Incorrect - may corrupt storage
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A contract stores user data in a dynamic array of structs (User[]).
2. The developer encodes and decodes the array using abi.encode/abi.decode.
3. The decoded result is reassigned to the original storage array.
4. Storage slot pointers are misaligned, causing the overwriting of other contract state.
5. This leads to corrupted user data, broken balances, or locked funds.

**Assumptions:**

- Contract uses Solidity <0.8.4 with ABIEncoderV2.
- Complex struct or nested array decoding is reassigned to storage.
- No version bump or manual storage mapping used to isolate data.

## ✅ Fixed Code

```solidity

// Upgrade to >=0.8.4 where ABIEncoderV2 is no longer experimental and stable
pragma solidity ^0.8.4;

contract SafeStorage {
    struct User {
        uint256 balance;
        string name;
    }

    User[] public users;

    function pushUser() public {
        users.push(User(100, "alice"));
    }

    // Use memory variables only when decoding
    function readUsers() external view returns (User[] memory) {
        return users;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid using ABIEncoderV2 for storage manipulation in versions <0.8.4.
- Upgrade to Solidity ≥0.8.4, where ABIEncoderV2 is the default and stable.
- Do not assign abi.decode results directly to storage variables.

### Additional Safeguards

- Use memory-local variables and intermediate buffers.
- Isolate storage from decode operations with dedicated update functions.
- Avoid nested struct encoding/decoding when referencing storage directly.

### Detection Methods

- Static analysis for abi.encode/abi.decode used on storage arrays.
- Manual review of contracts using pragma experimental ABIEncoderV2.
- Unit tests comparing expected vs actual storage after decode/assign.

## 🕰️ Historical Exploits

- **Name:** Solidity Storage Array Bugs 
- **Date:** June 2019 
- **Loss:** Incorrect data storage in signed integer arrays 
- **Post-mortem:** [Link to post-mortem](https://soliditylang.org/blog/2019/06/25/solidity-storage-array-bugs/)
  

## 📚 Further Reading

- [SWC-124: Arbitrary Location Write via Delegatecall](https://swcregistry.io/docs/SWC-124/) 
-  [Solidity Docs – Encoding and Decoding](https://docs.soliditylang.org/en/latest/abi-spec.html) 

  
## ✅ Vulnerability Report

```markdown
id: TBA
title: Storage ABIEncoderV2 Array Corruption
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 4  
complexity: 3     
detectability: 3  
finalScore: 3.6
```

---

## 📄 Justifications & Analysis

- **Impact**: Contract logic may break due to corrupted or overwritten storage.
- **Exploitability**: If storage is reassigned with abi-decoded data, no auth is needed.
- **Reachability**: Affects contracts that store complex structs or arrays.
- **Complexity**: Moderate—requires understanding of ABI layout and storage slots.
- **Detectability**: Easily missed unless ABI logic is explicitly reviewed.


