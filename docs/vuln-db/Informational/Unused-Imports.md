# Unused Imports

```YAML
id: LS04I
title: Unused Imports 
baseSeverity: I
category: code-quality
language: solidity
blockchain: [ethereum]
impact: Code bloat and possible audit confusion
status: draft
complexity: low
attack_vector: other
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-398
swc: SWC-103
```

## 📝 Description

- Unused imports refer to Solidity files or libraries that are included using import statements but never utilized within the contract. While not directly exploitable, these unused statements can lead to:
- Code bloat: Increased bytecode size if compiled improperly.
- Audit ambiguity: Auditors may waste time tracing unused libraries or functions.
- Increased maintenance burden: Future developers may misinterpret the purpose of unused imports.
- Although not an immediate vulnerability, removing such imports contributes to cleaner, more auditable, and secure code.

## 🚨 Vulnerable Code

```solidity

import "@openzeppelin/contracts/access/Ownable.sol"; // Unused
import "@openzeppelin/contracts/utils/math/SafeMath.sol"; // Used

contract Token {
    using SafeMath for uint256;
    
    uint256 public totalSupply;
}
```
## 🧪 Exploit Scenario

While there's no direct exploit:

1. Developer imports Ownable.sol assuming they may later use access control.
2. They forget to use Ownable or mistakenly think ownership logic is active.
3. Reviewers or future contributors are misled, wasting time or introducing errors.
4. If compiled incorrectly, bytecode may still include unused logic, bloating size.

**Assumptions:**

- Developers or auditors assume imported code is relevant.
- Compilation artifacts are not optimized to strip unused code.

## ✅ Fixed Code

```solidity

// Removed unused import
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract Token {
    using SafeMath for uint256;

    uint256 public totalSupply;
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "No exploit path, only affects clarity and code maintainability."
- context: "Mission-critical libraries with shadowed logic"
  severity: L
  reasoning: "Could mislead auditors or cause logic assumptions to fail."
- context: "Gas-optimized contracts"
  severity: I
  reasoning: "No impact if compiler strips unused imports during optimization."
```
## 🛡️ Prevention

### Primary Defenses

- Remove unused imports proactively during development.
- Adopt static analysis tools in CI pipelines.

### Additional Safeguards

- Enforce linting rules using tools like Solhint or ESLint.
- Use minimal contracts and libraries to reduce confusion.

### Detection Methods

- Slither Detects unused imports via solc AST.
- MythX May warn about unreferenced libraries.
- Manual Review Codebase walkthroughs help detect unused logic.

## 🕰️ Historical Exploits

- **Name:** OlympusDAO Range Bound Liquidity Bug
- **Date:** 2022 
- **Loss:** Found during audit
- **Post-mortem:** [Link to post-mortem](https://code4rena.com/audits/2022-08-olympus-dao-contest) 
  
## 📚 Further Reading

-  [SWC-131: Unused State Variables – SWC Registry](https://swcregistry.io/docs/SWC-131)
-  [Slither: Detect Unused Variables and Imports](https://github.com/crytic/slither#check-unused-vars-imports-functions) 

---

## ✅ Vulnerability Report

```markdown
id: LS04I
title: Unused Imports 
severity: I
score:
impact: 0         
exploitability: 0 
reachability: 5   
complexity: 0     
detectability: 5  
finalScore: 1.25
```

---

## 📄 Justifications & Analysis

- **Impact**: No effect on contract state or user funds.
- **Exploitability**: Cannot be directly used to compromise logic.
- **Reachability**: Unused imports are common and frequently left in codebases.
- **Complexity**: Easy to fix by removing the lines.
- **Detectability**: Readily detectable using static analysis tools like Slither or Solhint.

