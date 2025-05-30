# Missing NatSpec

```YAML
id: TBA
title: Missing NatSpec
baseSeverity: I
category: documentation
language: solidity
blockchain: [ethereum]
impact: Reduced code clarity and auditability
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-710
swc: SWC-103
```

## 📝 Description

- Constants in Solidity such as MAX, RATE, LIMIT, or ONE are frequently used without domain context, making their intent ambiguous. These names may:
- Represent drastically different things across contracts (supply cap vs. basis point divisor),
- Mislead during audits or integrations,
- Result in silent logic errors during forks or upgrades.
- Such misuse leads to bugs that often go unnoticed in security reviews and become integration hazards.

## 🚨 Vulnerable Code

```solidity

uint256 public constant MAX = 10000; // ❌ Meaning unclear

function calculateFee(uint256 amount) public pure returns (uint256) {
    return amount / MAX; // ❓ Is MAX basis points or something else?
}
```

## 🧪 Exploit Scenario

Step-by-step scenario:

1. A developer forks this code and assumes MAX refers to a MAX_SUPPLY value.
2. They use it in supply logic, not realizing it's a basis point divisor.
3. Fee math or cap enforcement breaks silently.
4. Users may overpay fees or receive invalid token amounts due to misaligned math.

**Assumptions:**

- Constant names are reused or inherited in unrelated contexts.
- No NatSpec or code comments clarify their role.

## ✅ Fixed Code

```solidity

uint256 public constant BASIS_POINT_DIVISOR = 10000;

/// @notice Calculates fee in basis points.
/// @param amount The amount to calculate the fee from.
/// @return The fee amount.
function calculateFee(uint256 amount) public pure returns (uint256) {
    return amount / BASIS_POINT_DIVISOR;
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "May introduce misunderstanding but not immediately exploitable."
- context: "Public forked protocol or token clone"
  severity: M
  reasoning: "Vague constants likely to cause silent logic errors during reuse."
- context: "Audited system with enforced naming conventions"
  severity: I
  reasoning: "Safe when naming conventions and static analysis are enforced."
```

## 🛡️ Prevention

### Primary Defenses

- Always use descriptive, domain-specific names (e.g., MAX_SUPPLY, FEE_RATE_DENOMINATOR).
- Document purpose and units using NatSpec on constants.

### Additional Safeguards

- Use module-specific constants to prevent cross-context misuse.
- Apply strict linting for naming conventions across teams and forks.

### Detection Methods

- Review for constants named MAX, ONE, RATE, LIMIT without clarifying suffixes or comments.
- Check whether such constants are reused across different functions or inherited contracts.
- Slither: naming-convention and constant-definition modules.

## 🕰️ Historical Exploits

- **Name:** Assert Violation in Solidity Smart Contracts
- **Date:** 2018–2023
- **Loss:** Estimated ~$150,000+ 
- **Post-mortem:** [Link to post-mortem](https://swcregistry.io/docs/SWC-110/)
  
## 📚 Further Reading

- [Solidity Documentation: Security Considerations](https://docs.soliditylang.org/en/latest/security-considerations.html)
- [Solidity — All About Errors Handling – Medium](https://jeancvllr.medium.com/solidity-all-about-errors-handling-99f7f02c17d)
- [Assert vs Require in Solidity – CodeForGeek](https://codeforgeek.com/assert-vs-require-in-solidity/)
- [MythX Smart Contract Weakness Classification – ConsenSys](https://consensys.io/mythx/detectors)
  
---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Ambiguous Constant Names
severity: I
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

- **Impact**: May cause silent misbehavior in financial calculations or limits.
- **Exploitability**: Vulnerable during forks or upgrades, not externally exploitable in isolation.
- **Reachability**: Constants are often used globally in calculations and modifiers.
- **Complexity**: Simple error; can be corrected with a naming update.
- **Detectability**: Easily caught via static analysis, linters, or naming guidelines.


