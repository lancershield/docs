# Redundant Code Logic 

```YAML

id: TBA
title: Redundant Code Logic Leading to Increased Gas Costs and Maintenance Risk
severity: L
category: code-quality
language: solidity
blockchain: [ethereum]
impact: Unnecessary gas costs, confusion, or potential hidden logic flaws
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-561
swc: SWC-131
```


## 📝 Description

- Redundant code logic refers to repeated, unnecessary, or unreachable lines of code that do not contribute to functional behavior but still:
- Consume gas on execution,
- Increase bytecode size,
- Lead to maintenance complexity or confusion during audits.
- Though not always a security threat, redundancy can hide subtle bugs or contradict the intended business logic when logic branches disagree.

## 🚨 Vulnerable Code

```solidity
contract RedundantExample {
    uint256 public value;

    function set(uint256 _value) public {
        value = _value;
        value = _value; // ❌ Redundant assignment

        if (_value > 0) {
            return;
        }
        return; // ❌ Unreachable duplicate return
    }
}
```

## 🧪 Exploit Scenario

1. This issue is not directly exploitable, but can lead to:
2. Higher deployment and execution costs (e.g., duplicate instructions),
3. Missed bugs if developers overlook one of two redundant lines thinking they serve different purposes,
4. Audit fatigue when redundant checks distract from true logic flaws.

**Assumptions:**

- Developers or tooling failed to clean up unnecessary logic paths.
- Contract is executed frequently, amplifying gas inefficiency.

## ✅ Fixed Code

```solidity

contract OptimizedExample {
    uint256 public value;

    function set(uint256 _value) public {
        value = _value;
        // ✅ Clean and clear
    }
}

```

## 🛡️ Prevention

### Primary Defenses

- Use static analyzers to detect duplicate lines, unreachable code, or repeated logic blocks.
- Apply code reviews and linters to flag unnecessary expressions or returns.
- Optimize for bytecode size and gas to reduce L1 fees.

### Additional Safeguards

- Write unit tests and gas snapshot tests to catch inefficient logic.
- Use DRY (Don’t Repeat Yourself) principles for internal business logic.
- Avoid mixing legacy code with new patterns unless refactored.

### Detection Methods

- Slither: redundant-statement, unused-return, dead-code detectors.
- Hardhat Gas Reporter: Highlight functions with excess cost.
- Manual audit and gas profiling of core functions.

## 🕰️ Historical Incidents

- **Name:** UniswapV1 Redundant Math in Slippage Check
-  **Date:** 2019 
-  **Impact:** Minor inefficiency and confusion in slippage calculation 
-  **Post-mortem:** [Link](https://uniswap.org/blog/uniswap-v1-launch/) 
  
## 📚 Further Reading

- [SWC-131: Presence of Unused Code](https://swcregistry.io/docs/SWC-131) 
- [Solidity Docs – Code Optimization](https://docs.soliditylang.org/en/latest/internals/optimizing-gas-costs.html) 
- [Slither – Gas and Code Quality Tools](https://github.com/crytic/slither) 


---
## ✅ Vulnerability Report

```markdown
id: TBA
title: Redundant Code Logic Leading to Increased Gas Costs and Maintenance Risk
severity: L
score:
impact: 2         
exploitability: 0 
reachability: 5   
complexity: 1     
detectability: 5  
finalScore: 2.1
```


---

## 📄 Justifications & Analysis

- **Impact**: Increases bytecode size and gas usage, may mislead audits.
- **Exploitability**: Not directly exploitable.
- **Reachability**: Extremely common during development cycles or copy-paste reuse.
- **Complexity**: Low — easy to write and easy to fix.
- **Detectability**: High — flagged by most static analysis tools.