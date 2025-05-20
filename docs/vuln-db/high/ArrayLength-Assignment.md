# Array Length Assignment Can Cause Storage Corruption or Denial of Service

```YAML
id: TBA
title: Array Length Assignment Can Cause Storage Corruption or Denial of Service
severity: H
category: storage
language: solidity
blockchain: [ethereum]
impact: Unbounded gas usage, memory corruption, or state overwrite
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.6.0"]
cwe: CWE-119
swc: SWC-128
```

## 📝 Description

- In older versions of Solidity (<0.6.0), dynamic arrays in storage allowed direct assignment to the .
- length property, which was unsafe and could be abused to manipulate internal memory or storage slots. 
- This could cause:Overwrites of adjacent storage slots.
- Unexpected gas exhaustion from large allocations.
- Critical logic failures from invalid array index assumptions.
- Starting in Solidity 0.6.0, direct assignments to .length on storage arrays have been deprecated in favor of using push() and pop() methods, preventing accidental or malicious memory control.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.5.17;

contract DangerousArray {
    uint256[] public data;

    function setLength(uint256 newLength) public {
        data.length = newLength; // ❌ Unsafe in older Solidity
    }

    function writeData(uint256 index, uint256 value) public {
        data[index] = value;
    }
}
```

## 🧪 Exploit Scenario

1. Attacker calls setLength(2**256 - 1) to extend the array length to near-maximum.
2. The next call to writeData(index, value) with a large index writes far outside the intended bounds.
3. The attacker may overwrite storage slots like owner, balances, or other critical variables depending on layout.
4. Alternately, long iterations or processing based on data.length can be made to consume excessive gas.

**Assumptions:**

- Contract uses Solidity <0.6.0 and does not validate input to .length assignment.
- The array is used in logic that relies on .length or is iterated over.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeArray {
    uint256[] public data;

    function append(uint256 value) external {
        data.push(value); // ✅ Safe length manipulation
    }

    function truncate(uint256 newLength) external {
        require(newLength <= data.length, "Cannot increase length");
        while (data.length > newLength) {
            data.pop(); // ✅ Controlled shrink
        }
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use Solidity ≥0.6.0 to disallow direct .length assignments on storage arrays.
- Remove functions that expose .length as a writable value.

### Additional Safeguards

- Validate index bounds before writing to any array element.
- Avoid exposing raw storage arrays to external mutation.
- Use fixed-size arrays where feasible.

### Detection Methods

- Grep/static analysis for .length = in contracts using older Solidity.
- Review all functions that mutate dynamic arrays.
- Tools: Slither (dangerous-array-length), MythX, manual inspection

## 🕰️ Historical Exploits

- **Name**: Solidity 0.5.x Array Length Abuse
- **Date**: 2019
- **Loss**: No direct loss, but improper array length usage led to logic errors in production systems
- **Post-mortem**: [Link to post-mortem](https://docs.soliditylang.org/en/v0.6.0/060-breaking-changes.html#explicitness-requirements) 


## 📚 Further Reading

- [SWC-128: DoS with Block Gas Limit](https://swcregistry.io/docs/SWC-128/)  
- [Ethereum StackExchange: Is setting array length dangerous?](https://ethereum.stackexchange.com/questions/25460/solidity-dynamic-array-set-length) 
- [OpenZeppelin – Safe Patterns for Array Management](https://docs.openzeppelin.com/contracts/4.x/api/utils#Arrays) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Array Length Assignment Can Cause Storage Corruption or Denial of Service
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 3   
complexity: 3     
detectability: 3  
```

---

## 📄 Justifications & Analysis

- **Impact**: Dangerous .length changes can lead to memory corruption or infinite gas loops.
- **Exploitability**: Straightforward if setter is public or accessible.
- **Reachability**: Commonly exposed via admin or upgrade functions in legacy systems.
- **Complexity**: Requires moderate understanding of Solidity storage layout.
- **Detectability**: Subtle bug; easy to overlook unless reviewing pre-0.6.0 Solidity carefully.







