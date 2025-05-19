# Incorrect Shift in Assembly

```YAML 
id: TBA
title: Incorrect Shift in Assembly Causes Arithmetic Errors and State Corruption
severity: H
category: arithmetic
language: solidity
blockchain: [ethereum]
impact: Logic failure, value truncation, or privilege escalation
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-682
swc: SWC-128
```

## 📝 Description

- In Solidity inline assembly, bitwise shift operations (shl, shr, sar) are low-level instructions that manipulate binary data directly. 
- An incorrect shift operation—such as shifting by the wrong number of bits, using signed vs unsigned incorrectly, or applying shifts to unbounded data—can lead to logic errors, truncated values, or corrupted state.
- This vulnerability typically arises when using shift-based encoding/decoding, packing/unpacking variables, or interpreting calldata and storage manually. 
- Improperly shifted bits may result in wrong address values, token amounts, permissions, or storage slot resolutions—causing denial of service, invalid memory writes, or security bypasses.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract ShiftBug {
    function pack(uint128 high, uint128 low) public pure returns (uint256 result) {
        assembly {
            result := or(shl(128, high), low) // ✅ Correct shift is 128 bits
        }
    }

    function bug(uint128 high, uint128 low) public pure returns (uint256 result) {
        assembly {
            result := or(shl(120, high), low) // ❌ Incorrect: shift by 120 instead of 128
        }
    }
}
```

## 🧪 Exploit Scenario

1. A protocol uses pack() to store two values (e.g., price and quantity) into a single uint256.
2. A developer mistakenly uses shl(120, high) instead of shl(128, high).
3. When the packed value is decoded or used for logic, bits from high leak into low.
4. This alters permissions, breaks math, or leads to token miscounting.

**Assumptions:**

- Assembly is used to optimize storage or calldata.
- Shifts are not precisely calculated or reviewed.
- Packed values are used in sensitive logic (permissions, amounts, identifiers).

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract ShiftFix {
    function pack(uint128 high, uint128 low) public pure returns (uint256 result) {
        assembly {
            result := or(shl(128, high), low) // ✅ Correct: 128-bit boundary
        }
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid inline assembly unless absolutely necessary.
- Use Solidity’s native packing (e.g., abi.encodePacked) for simple tasks.
- When using shl or shr, match the shift amount exactly to the type width.

### Additional Safeguards

- Use constants or enums to define shift values.
- Write unit tests for packing/unpacking correctness.
- Consider writing custom pack()/unpack() helper libraries in Solidity.

### Detection Methods

- Review shift amounts in shl, shr, sar.
- Check for shift overflows, underflows, or mismatches with variable sizes.
- Tools: Slither, Manticore, manual assembly audit

## 🕰️ Historical Exploits

- **Name:** Shift Operation Misuse in Smart Contract 
- **Date:** 2022 
- **Loss:** Logical errors leading to incorrect contract behavior
- **Post-mortem:** [Link to post-mortem](https://ethereum.stackexchange.com/questions/127538/right-shift-not-working-in-inline-assembly) -
 
## 📚 Further Reading

- [SWC-128: DoS With Block Gas Limit](https://swcregistry.io/docs/SWC-128/) 
- [Solidity Inline Assembly Guide](https://docs.soliditylang.org/en/latest/assembly.html) 
- [Yul and Optimizing Shift Operations](https://ethereum.stackexchange.com/questions/96672/solidity-assembly-why-choose-shl-over-mul) 

---
  
## ✅ Vulnerability Report 
```markdown
id: TBA
title: Incorrect Shift in Assembly Causes Arithmetic Errors and State Corruption
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 3   
complexity: 4     
detectability: 3  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Can corrupt key values like token amounts, permission bits, or packed addresses.
- **Exploitability**: Requires a precise misuse in assembly; attacker can exploit if inputs affect shift targets.
- **Reachability**: Assembly is rare but often used in core protocol or gas-optimized paths.
- **Complexity**: Shifting by wrong bit count is a common mistake; math knowledge required.
- **Detectability**: Hidden unless logic is deeply understood or coverage testing is thorough.

