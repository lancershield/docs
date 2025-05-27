# Dangerous Enum Conversion

```YAML
id: TBA
title: Dangerous Enum Conversion 
baseSeverity: M
category: type-conversion
language: solidity
blockchain: [ethereum]
impact: Incorrect branching logic, privilege escalation, or execution bypass
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-686
swc: SWC-136
```

## 📝 Description

- Solidity enums are internally stored as uint8 values, where each enum member maps to a sequential number (starting at 0). However, if a uint is cast to an enum without proper bounds checks, it can result in invalid enum values that are not defined in the type declaration. 
- This leads to undefined behavior such as:
- Executing the wrong case in a switch/if block
- Skipping default logic due to unreachable branches
- Allowing an attacker to escalate privileges or bypass checks
- This issue typically occurs when enums are deserialized from external data (e.g., uint256 input from calldata or storage) and cast directly into the enum type without validation.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract EnumExample {
    enum Role { None, User, Admin }

    function execute(uint8 roleRaw) external {
        Role role = Role(roleRaw); // ❌ Dangerous: No bounds check

        if (role == Role.Admin) {
            // privileged action
        } else if (role == Role.User) {
            // limited access
        } else {
            // assume Role.None
        }
    }
}
```

## 🧪 Exploit Scenario

1. A contract uses Role enum to control access.
2. An attacker calls execute(99) to pass an invalid role.
3. Since Role(99) is technically valid (as a uint8), it bypasses all defined checks.
4. The attacker exploits logic gaps or executes code meant to be unreachable.

**Assumptions:**

- Contract converts raw integers to enums without validation.
- Conditional logic relies on specific enum branches.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeEnumExample {
    enum Role { None, User, Admin }

    function execute(uint8 roleRaw) external {
        require(roleRaw <= uint8(Role.Admin), "Invalid role");
        Role role = Role(roleRaw);

        if (role == Role.Admin) {
            // privileged action
        } else if (role == Role.User) {
            // limited access
        } else {
            // default behavior
        }
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Enum governs access control or phase-critical logic"
  severity: M
  reasoning: "May bypass security checks or trigger wrong state logic"
- context: "Enum values stored or passed externally"
  severity: H
  reasoning: "Attacker may inject invalid enum states"
- context: "Enum fully internal and validated on every assignment"
  severity: I
  reasoning: "No exposure if casting is controlled and bounded"
```

## 🛡️ Prevention

### Primary Defenses

- Never cast unchecked integers to enums.
- Always validate numeric input is within the enum's range.

### Additional Safeguards

- Use constants or libraries to define MAX_ENUM_VALUE.
- Use OpenZeppelin's AccessControl or role-based guards instead of custom enums for access management.

### Detection Methods

- Look for MyEnum(var) casts from uint, uint8, etc.
- Check whether numeric inputs are validated before enum conversion.
- Tools: Slither (enum-conversion), manual type safety review

## 🕰️ Historical Exploits

- **Name:** Solidity Enum Conversion Bug 
- **Date:** 2016 
- **Loss:** Potential for unintended behavior due to out-of-range enum values 
- **Post-mortem:** [Link to post-mortem](https://coinsbench.com/smart-contract-vulnerabilities-3-401a05e94078) 


## 📚 Further Reading

- [SWC-136: Unencrypted Sensitive Data (related to unsafe type use)](https://swcregistry.io/docs/SWC-136/)
- [Solidity Docs – Enum Type](https://docs.soliditylang.org/en/latest/types.html#enums)  
- [Best Practices – Enum Safety](https://ethereum.stackexchange.com/questions/117662/how-do-i-safely-cast-enums-in-solidity) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Dangerous Enum Conversion
severity: M
score:
impact: 4         
exploitability: 3 
reachability: 3   
complexity: 2     
detectability: 4 
finalScore: 3.45
```

---

## 📄 Justifications & Analysis

- **Impact**: Can corrupt access control logic, allow privilege escalation, or break invariants.
- **Exploitability**: Requires passing a valid uint that maps to undefined enum value.
- **Reachability**: Common in public interfaces and permissioned logic paths.
- **Complexity**: Not difficult to exploit once the pattern is known.
- **Detectability**: Dangerous due to subtlety; missed unless explicitly checking enum boundaries.