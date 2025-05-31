# Unsorted Imports

```YAML
id: LS13I
title: Unsorted Imports
baseSeverity: I
category: code-style
language: solidity
blockchain: [ethereum]
impact: Reduced code readability and auditability
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.6.0", "<0.8.24"]
cwe: CWE-563
swc: SWC-135
```

## 📝 Description

- Unsorted or inconsistent import statements—such as unordered library imports, unused imports mixed with critical ones, or grouping unrelated imports together—can reduce code readability, hinder static analysis, and lead to confusion for auditors and developers.
- Although not directly exploitable, disorganized imports contribute to maintenance overhead and auditing fatigue, potentially delaying the discovery of real vulnerabilities.

## 🚨 Vulnerable Code

```solidity

import "./interfaces/IUniswapV2Router.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "./MyToken.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

## 🧪 Exploit Scenario

Step-by-step mismanagement outcome:

1. A developer or auditor scans the contract.
2. Disorganized imports cause confusion about which modules are used.
3. An unused import is mistaken for a real dependency, diverting attention.
4. A critical module might be overlooked, especially in large contracts.
5. Slower audits increase the chance of missing genuine vulnerabilities.

**Assumptions:**

- Applies during development, review, or audit stages. No attacker involvement necessary.

## ✅ Fixed Code

```solidity

// Grouped and sorted imports
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

import "./MyToken.sol";
import "./interfaces/IUniswapV2Router.sol";
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "No security risk; affects only readability and audit speed."
- context: "Large Protocol Codebase with Multiple Auditors"
  severity: L
  reasoning: "Increases cognitive load and introduces review errors in multi-file audits."
- context: "Minimal Contract with Few Imports"
  severity: I
  reasoning: "No meaningful risk or confusion in small contracts."
```

## 🛡️ Prevention

### Primary Defenses

- Use alphabetical ordering or grouping by source (e.g., third-party vs local).
- Adopt automated formatting with tools like Prettier, Hardhat plugins, or Foundry's forge fmt.

### Additional Safeguards

- Linting rules (e.g., Solhint ordering) to enforce import sort order.
- CI pipelines rejecting unformatted or disordered code submissions.

### Detection Methods

- Solhint with import order enforcement.
- Prettier Plugin for Solidity

## 🕰️ Historical Exploits

-  **Name:** Uniswap v1 Fork Confusion Incident 
-  **Date:** 2020 
-  **Loss:** None 
-  **Post-mortem:** [Link to post-mortem](https://consensys.net/diligence/) 
  
## 📚 Further Reading

- [SWC-135: Code With No Effects – SWC Registry](https://swcregistry.io/docs/SWC-135/)
- [Prettier Solidity Plugin](https://github.com/prettier-solidity/prettier-plugin-solidity)
  
---

## ✅ Vulnerability Report 

```markdown
id: LS13I
title: Unsorted Imports
severity: I
score:
impact: 0
exploitability: 0
reachability: 5
complexity: 1
detectability: 5
finalScore: 1.15
```

---

## 📄 Justifications & Analysis

- **Impact**: No state or financial risk introduced by disordered imports.
- **Exploitability**: Cannot be exploited by a malicious party.
- **Reachability**: Every contract with imports can be affected.
- **Complexity**: Easy to understand and fix.
- **Detectability**: Obvious during code review or with automated tools.