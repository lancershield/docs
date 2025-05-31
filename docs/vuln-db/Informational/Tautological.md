# Tautological

```YAML
id: LS03I
title: Tautological 
baseSeverity: I
category: logic
language: solidity
blockchain: [ethereum]
impact: Ineffective checks, audit confusion, gas waste
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.6.0"]
cwe: CWE-570
swc: SWC-131
```

## 📝 Description

- A tautological compare occurs when a condition always evaluates to true or false due to the way values are constrained or types are defined.
- In Solidity, this typically happens when:
- Unsigned integers (uint) are compared to < 0 (always false)
- Boolean values are checked redundantly (e.g., if (flag == true))
- Enum values are checked against impossible conditions
- Comparisons rely on constant return values or static inputs
- These conditions lead to unreachable code, unnecessary gas usage, or logic branches that mislead reviewers, potentially obscuring genuine security-relevant logic.

## 🚨 Vulnerable Code

```solidity
pragma solidity ^0.8.0;

contract TautologyExample {
    function isValid(uint256 amount) public pure returns (bool) {
        if (amount >= 0) { // ❌ Always true for uint256
            return true;
        } else {
            return false;
        }
    }
}
```

## 🧪 Exploit Scenario

1. A developer writes a check like if (amount >= 0) intending to guard input.
2. They add different behaviors to the if and else branches.
3. However, the else branch can never execute.
4. This may lead to incorrect assumptions about coverage, testing, or validation logic.

**Assumptions:**

- The code uses unsigned types or constants.
- Developer unintentionally introduces an unreachable path.

## ✅ Fixed Code

```solidity
pragma solidity ^0.8.0;

contract FixedCompare {
    function isValid(uint256 amount) public pure returns (bool) {
        // Just return true, since uint >= 0 is always true
        return true;
    }
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: I
  reasoning: "No direct impact, but adds noise to logic and gas usage."
- context: "High-frequency DeFi function"
  severity: M
  reasoning: "Can lead to measurable gas inefficiency at scale."
- context: "Testnet or internal utility contract"
  severity: I
  reasoning: "Negligible impact when not used in production flows."
```

## 🛡️ Prevention

### Primary Defenses

- Avoid comparing unsigned integers against 0 with >= or < (redundant).
- Use direct boolean expressions (if (flag) instead of if (flag == true)).

### Additional Safeguards

- Run static analysis to catch unreachable code.
- Add unit tests that explicitly check for all logic branches.

### Detection Methods

- Search for tautologies like uint >= 0, x == true, false == false, etc.
- Tools: Slither (tautology, dead-code), Surya, manual review

## 🕰️ Historical Exploits

- **Name:** Solidity Tautological Comparison Issue 
- **Date:** 2022 
- **Loss:** Potential for unintended behavior due to always-true conditions 
- **Post-mortem:** [Link to post-mortem](https://medium.com/@bartubozkurt35/smart-contract-vulnerabilities-2-de08d0ac73c2) 
  
## 📚 Further Reading

- [SWC-135: Incorrect Logic](https://swcregistry.io/docs/SWC-135/) 
- [Slither Tautology Detector](https://github.com/crytic/slither/wiki/Detector-Documentation#tautologies) 
- [Solidity Docs – Comparison Operators](https://docs.soliditylang.org/en/latest/control-structures.html#comparison-operators)
- [Trail of Bits – Secure Smart Contract Patterns](https://github.com/crytic/building-secure-contracts) 

---

## ✅ Vulnerability Report

```markdown
id: LS03I
title: Tautological 
severity: I
score:
impact: 2         
exploitability: 1 
reachability: 3  
complexity: 1     
detectability: 5  
finalScore: 2.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Logic is misleading or ineffective, but not directly exploitable in most cases.
- **Exploitability**: Cannot be abused unless misused in access control or conditional logic.
- **Reachability**: Commonly encountered in code with type misunderstandings.
- **Complexity**: A trivial but easy-to-make mistake.
- **Detectability**: Very easy to detect with static analysis or basic code review.
