# Ambiguous Constant Names

```YAML
id: TBA
title: Ambiguous Constant Names 
severity: L
category: naming
language: solidity
blockchain: [ethereum]
impact: Misinterpreted logic, developer error, or configuration mistakes
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-665
swc: SWC-135
```

## 📝 Description

- Poorly named or ambiguous constants (e.g., MAX, ONE, RATE, PERCENTAGE) can introduce logical bugs and misinterpretation during both development and auditing.
- These issues are particularly dangerous when constants are reused across unrelated domains or are assumed to represent standard values (e.g., 100 vs 10,000 basis points).
- This vulnerability doesn't lead to direct exploits but often results in:
- Incorrect math logic
- Mistuned parameters
- Broken thresholds or caps
- Unintended privilege escalation or overly strict checks
- If constants like MAX, MIN, or DENOMINATOR are poorly named or reused across different contexts, downstream developers or auditors may assume incorrect behavior.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract AmbiguousConstants {
    uint256 public constant MAX = 10000;  // ❌ Is this basis points, percentage, or token cap?
    uint256 public fee = 250;             // 2.5% if MAX is basis points, but not obvious

    function getFee(uint256 amount) external view returns (uint256) {
        return (amount * fee) / MAX;
    }
}
```

## 🧪 Exploit Scenario

1. A developer forks the contract and changes MAX = 100, assuming it's a percentage denominator.
2. The fee is left at 250, which now implies a 250% fee, not 2.5%.
3. The deployed contract silently overcharges users.
4. Alternatively, an auditor assumes MAX means the token cap and misses a privilege violation tied to fee logic.

**Assumptions:**

- Constants are globally defined with unclear or reused names.
- No comments or context help distinguish meaning.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract ClearConstants {
    uint256 public constant BASIS_POINTS_DIVISOR = 10000; // ✅ self-documenting
    uint256 public feeBps = 250; // 2.5%

    function getFee(uint256 amount) external view returns (uint256) {
        return (amount * feeBps) / BASIS_POINTS_DIVISOR;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Name constants descriptively to reflect their domain (_BPS, _LIMIT, _DENOMINATOR, _SECONDS, etc.).
- Include inline comments for all public constants.

### Additional Safeguards

- Group constants by purpose or feature domain (e.g., fee settings, time delays).
- Use immutable constructor parameters if values differ across environments.

### Detection Methods

- Look for constants with names like MAX, RATE, ONE, LIMIT, without domain qualifiers.
- Review math logic that depends on unclear denominators.
- Tools: Custom linters, Slither (naming-convention), manual audit

## 🕰️ Historical Exploits

- **Name:** Forked Token Misconfigured Fee Rate 
- **Date:** 2021 
- **Loss:** Contract charged excessive fees due to misinterpreted `MAX` constant 
- **Post-mortem:** [Link to post-mortem](https://web.archive.org/web/*/redacted.example.com) 
  

## 📚 Further Reading

- [SWC-135: Incorrect Logic](https://swcregistry.io/docs/SWC-135/) 
- [Solidity Naming Conventions](https://docs.soliditylang.org/en/latest/style-guide.html#naming-conventions)
- [Trail of Bits: Common Solidity Misunderstandings](https://blog.trailofbits.com/) 
  
---
  
## ✅ Vulnerability Report
```markdown
id: TBA
title: Ambiguous Constant Names 
severity: L
score:
impact: 2  
exploitability: 2 
reachability: 4 
complexity: 1  
detectability: 5 
finalScore: 2.4
```

---

## 📄 Justifications & Analysis

- **Impact**: Logic tied to vague constants may break silently or mislead audits.
- **Exploitability**: Error-prone for forks, integrations, or upgrades.
- **Reachability**: Global constants affect fee math, limits, durations, etc.
- **Complexity**: Trivial—resolved with better naming.
- **Detectability**: Easily caught with naming guidelines or visual inspection.