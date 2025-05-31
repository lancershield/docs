# Inconsistent Style

```YAML
id: LS11I
title: Inconsistent Style
baseSeverity: I
category: readability
language: solidity
blockchain: [ethereum]
impact: Reduces auditability and increases misinterpretation risk
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.23"]
cwe: CWE-710
swc: SWC-135
```

## 📝 Description

- Inconsistent code style in smart contracts—such as variable naming mismatches, inconsistent indentation, or unpredictable ordering of functions—does not directly lead to exploitation, but significantly hinders code comprehension, increases the chance of logic misunderstandings, and slows down the audit process.
- More dangerously, inconsistent style can mask subtle bugs (e.g., misleading variable names or out-of-place logic) and lead to approval of flawed logic during human or automated review. 
- This is especially critical in governance or initialization functions, where visual cues are often the first line of defense.

## 🚨 Vulnerable Code

```solidity

// Different naming styles in the same contract
uint userBalance;
mapping(address => uint) balances;

function DepositETH() public payable { // PascalCase instead of camelCase
    balances[msg.sender] += msg.value;
}

function withdraw_eth(uint amount) public { // snake_case
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount;
    (bool sent, ) = msg.sender.call{value: amount}("");
    require(sent);
}
```

## 🧪 Exploit Scenario

Step-by-step issue analysis:

1. A developer or auditor misreads userBalance as a distinct variable, not realizing it refers to balances[msg.sender].
2. Function names follow inconsistent casing and terminology, making logic hard to trace across files or interfaces.
3. Tooling like auto-doc generators or interfaces may generate confusing or incomplete ABI.
4. Auditor misses a misordered logic bug in withdraw_eth due to style fatigue and poor visual hierarchy.

**Assumptions:**

- No enforced linting in CI/CD pipeline.
- Multideveloper codebase with no agreed style guide.

## ✅ Fixed Code

```solidity

// Consistent naming and style
mapping(address => uint256) userBalances;

function depositEth() public payable {
    userBalances[msg.sender] += msg.value;
}

function withdrawEth(uint256 amount) public {
    require(userBalances[msg.sender] >= amount, "Insufficient balance");
    userBalances[msg.sender] -= amount;
    (bool sent, ) = msg.sender.call{value: amount}("");
    require(sent, "ETH transfer failed");
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "Affects maintainability and audit clarity but not runtime behavior."
- context: "DeFi Protocol with complex logic"
  severity: L
  reasoning: "Increases audit time and risk of overlooking subtle logic flaws."
- context: "Single-developer experimental contract"
  severity: I
  reasoning: "Limited scope, lower impact despite poor readability."
```

## 🛡️ Prevention

### Primary Defenses

- Enforce project-wide style guides (e.g., Solidity Style Guide, Airbnb-like rules).
- Use consistent naming, spacing, and function ordering.

### Additional Safeguards

- Implement autoformatters and linting tools (e.g., Prettier for Solidity, Solhint).
- Conduct code reviews with style consistency as a checklist item.

### Detection Methods

- Run solhint with rules like func-name-mixedcase, no-multiple-empty-lines, no-unused-vars.
- Use GitHub Actions to block inconsistent code from merging.
- Slither’s naming-convention detector also helps identify style mismatches.

## 🕰️ Historical Exploits

- **Name:** DAO Contract Vulnerability  
- **Date:** 2016  
- **Loss:** ~$60  
- **Post-mortem:** [Link to post-mortem](https://codeofcode.org/lessons/case-studies-of-real-world-smart-contract-vulnerabilities-and-exploits/)  


## 📚 Further Reading

- [SWC-135: Code With No Effects – SWC Registry](https://swcregistry.io/docs/SWC-135/) 
- [Solidity Style Guide – Official](https://docs.soliditylang.org/en/latest/style-guide.html) 
- [Slither: Solidity Static Analyzer](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report

```markdown
id: LS11I
title: Inconsistent Style
severity: I
score:
impact: 0
exploitability: 0
reachability: 5
complexity: 1
detectability: 5
finalScore: 0.85
```

---

## 📄 Justifications & Analysis

- **Impact**: No direct functional impact or loss potential.
- **Exploitability**: Cannot be exploited in runtime behavior.
- **Reachability**: Common in almost all developer-written codebases.
- **Complexity**: Easy to introduce; minimal effort or setup needed.
- **Detectability**: Highly visible with both manual and automated tools