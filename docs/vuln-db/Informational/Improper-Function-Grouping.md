# Improper Function Grouping

```YAML
id: LS10I
title: Improper Function Grouping
baseSeverity: I
category: readability
language: solidity
blockchain: [ethereum]
impact: Misinterpretation and maintenance risks
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">0.6.0", "<0.8.20"]
cwe: CWE-710
swc: SWC-135
```

## 📝 Description

- Improper function grouping occurs when related functions are scattered across the contract rather than logically ordered together. 
- This issue doesn’t pose a direct security threat but increases cognitive load, reduces maintainability, and may mislead auditors or developers—potentially masking critical logic flaws or violating intended access controls.
- Grouping related functions (e.g., external interfaces, internal helpers, modifiers) improves readability, auditing efficiency, and reduces risk of errors in large codebases.

## 🚨 Vulnerable Code

```solidity

contract Voting {
    function vote(uint256 id) external { /* voting logic */ }

    function _resetVotes() internal { /* reset logic */ }

    function getVoteCount(uint256 id) public view returns (uint256) { /* getter */ }

    function _internalLogic1() internal { /* helper */ }

    function initialize() public { /* setup */ }

    function _internalLogic2() internal { /* another helper */ }
}
```

## 🧪 Exploit Scenario

1. Auditor reviews the initialize() function but overlooks _internalLogic2() as it's defined far apart.
2. Developer mistakenly adds new logic assuming only _resetVotes() is involved post-initialization.
3. Hidden state changes in _internalLogic2() go unnoticed during upgrades or code changes.

**Assumptions:** 

- Large contracts with mixed public/private/internal methods interleaved without logical separation.

## ✅ Fixed Code

```solidity

contract Voting {
    // === External/Public Functions ===
    function vote(uint256 id) external { /* voting logic */ }
    function getVoteCount(uint256 id) public view returns (uint256) { /* getter */ }
    function initialize() public { /* setup */ }

    // === Internal Helper Functions ===
    function _resetVotes() internal { /* reset logic */ }
    function _internalLogic1() internal { /* helper */ }
    function _internalLogic2() internal { /* another helper */ }
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: L
  reasoning: "Poses readability and auditability risk, but no direct user impact."
- context: "Public DeFi Protocol undergoing audits"
  severity: M
  reasoning: "Poor grouping could obscure critical function flows, risking auditor oversight."
- context: "Simple internal-only contract"
  severity: I
  reasoning: "Low developer count and minimal external interface reduces any risk of confusion."
```

## 🛡️ Prevention

### Primary Defenses

- Group functions by visibility: external/public first, internal/private later.
- Cluster related logic (e.g., setup functions, state modifiers) together.

### Additional Safeguards

- Use NatSpec for logical grouping and section documentation.
- Enforce ordering through linting rules (e.g., Solhint, Prettier Solidity).

### Detection Methods

- Code review with structure guidelines.
- Static analysis tools to check ordering, e.g., Slither custom detectors.

## 🕰️ Historical Exploits
 
- **Name:** Parity Wallet Multisig Bug  
- **Date:** 2017  
- **Loss:** ~$30  
- **Post-mortem:** [Link to post-mortem](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7)  
- **Name:** Bancor Network Hack  
- **Date:** 2018  
- **Loss:** ~$23.5 million  
- **Post-mortem:** [Link to post-mortem](https://techcrunch.com/2018/07/10/bancor-loses-23-5m/)

## 📚 Further Reading

- [SWC-100: Function Default Visibility – SWC Registry](https://swcregistry.io/docs/SWC-100/)   
- [Consensys Diligence: Smart Contract Security Best Practices](https://diligence.consensys.io/categories/best-practice/)

## ✅ Vulnerability Report

```markdown
id: LS10I
title: Improper Function Grouping
severity: I
score:
impact: 1
exploitability: 0
reachability: 5
complexity: 1
detectability: 4
finalScore: 1.45
```

---

## 📄 Justifications & Analysis

- **Impact**: No direct security risk, but may lead to bugs during upgrades or misunderstanding of logic.
- **Exploitability**: No active exploit possible.
- **Reachability**: Readability impacts every developer and reviewer who interacts with the contract.
- **Complexity**: Identifying disordered functions is trivial.
- **Detectability**: Easily detectable via code review and linters, but often ignored.

