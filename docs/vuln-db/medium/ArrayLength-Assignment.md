# Array Length Assignment

```YAML
id: TBA
title: Array Length Assignment
baseSeverity: M
category: memory-corruption
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Memory corruption, data exposure, or unintended out-of-bounds behavior
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.6.0"]
cwe: CWE-119
swc: SWC-128
```

## 📝 Description

- Prior to Solidity 0.6.0, it was possible to assign directly to the length of dynamic storage or memory arrays using array.length = newLength. 
- This operation could be abused to shrink or expand arrays improperly, causing unintended behavior such as:
- Out-of-bounds access
- Overwriting existing memory/state
- Erasing elements from the array without emitting events or cleanup
- In newer Solidity versions (>=0.6.0), length assignment is deprecated or disallowed, but legacy contracts or forked compilers might still include this pattern.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.5.17;

contract ArrayExample {
    uint[] public data;

    function truncate() public {
        data.length = 1; // ❌ direct length assignment
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A protocol stores critical configuration or user balances in a dynamic array.
2. Admin or attacker contract calls a function that sets data.length = 0.
3. This wipes out all existing elements without a proper reset or record.
4. If the contract relies on data.length for validation, subsequent loops may behave incorrectly.
5. Users may lose access to stored values or logic may process empty arrays incorrectly.

**Assumptions:**

- The contract uses Solidity <0.6.0.
- length assignment is permitted and not restricted by modifiers.
- The array in question is tied to sensitive data or control flows.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeArray {
    uint[] public data;

    function clearArray() public {
        delete data; // ✅ preferred way to reset dynamic arrays
    }

    function removeLast() public {
        require(data.length > 0, "Empty array");
        data.pop(); // ✅ safe element removal
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Legacy contract with length mutation on user funds or indexes"
  severity: M
  reasoning: "Can result in silent loss of critical state or skipped validations."
- context: "Modern contract using delete/pop with bounds checks"
  severity: I
  reasoning: "No practical risk with modern practices."
- context: "Array is internal and cleared during safe lifecycle events"
  severity: L
  reasoning: "Minimal risk if use is predictable and controlled."
```

## 🛡️ Prevention

### Primary Defenses

- Use Solidity ≥0.6.0 which deprecates direct length assignment.
- Prefer delete array or controlled pop() loops for truncation.
- Validate array length before operations (require(array.length > 0)).

### Additional Safeguards

- Emit events when truncating or modifying arrays
- Use mappings instead of arrays for indexed user data where applicable
- Perform extensive testing for array edge cases (empty, 1 element, max size)

### Detection Methods

- Slither: Custom rule to detect array.length = assignment
- Manual audit of storage array usage in Solidity <0.6.0
- Linter rules for unsafe array length operations

## 🕰️ Historical Exploits

- **Name:** Paradigm CTF Bank Challenge
- **Date:** 2021
- **Loss:** Simulated exploit in CTF environment 
- **Post-mortem:** [Link to post-mortem](https://smarx.com/posts/2021/02/writeup-of-paradigm-ctf-bank/)
  
## 📚 Further Reading

- [SWC-128: DoS With Block Gas Limit (Loop Overrun)](https://swcregistry.io/docs/SWC-128) 
- [CWE-119: Improper Restriction of Operations within Memory Buffer](https://cwe.mitre.org/data/definitions/119.html) 
- [Solidity v0.6.0 Changelog – Array Length Assignment Removed](https://github.com/ethereum/solidity/releases/tag/v0.6.0)
- [Solidity Docs – Arrays](https://docs.soliditylang.org/en/latest/types.html#arrays) 
  
---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Array Length Assignment
severity: M
score:
impact: 3
exploitability: 3
reachability: 3
complexity: 2
detectability: 4
finalScore: 3.05
```

---

## 📄 Justifications & Analysis

- **Impact**: Allows silent data deletion or unexpected behavior in control flows using dynamic arrays.
- **Exploitability**: Trivial in older Solidity versions; requires only one line of code.
- **Reachability**: Often used in token lists, governance queues, or balance records.
- **Complexity**: Low — a single call can trigger the issue, especially in unguarded contexts.
- **Detectability**: Easy to catch via static analysis or code review in older Solidity codebases.