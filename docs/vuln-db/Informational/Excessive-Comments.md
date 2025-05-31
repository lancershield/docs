# Excessive Comments

```YAML
id: LS09I
title: Excessive Comments 
baseSeverity: I
category: code-quality
language: solidity
blockchain: [ethereum]
impact: Code bloat and possible leakage of internal logic
status: draft
complexity: low
attack_vector: other
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.23"]
cwe: CWE-615
swc: SWC-136
```

## 📝 Description

- Excessive or verbose comments in Solidity source code—especially when compiled with metadata embedded—can lead to unnecessarily large bytecode deployments and leakage of sensitive logic. While comments themselves do not execute, when metadata hashes or flattened contracts are published, these comments can expose internal reasoning, assumptions, or even admin details unintentionally.
- Though primarily a code hygiene issue, in some circumstances this may provide breadcrumbs to exploit logic or assist in social engineering attacks.

## 🚨 Vulnerable Code

```solidity
// It was initially written by Alice, later modified by Bob and then again changed...
function transfer(address to, uint256 amount) public {
    balances[msg.sender] -= amount;
    balances[to] += amount;
}
```
## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Contract is published with source code and full comments to a block explorer.
2. Adversary scrapes contract comments via verified source or metadata.
3. Extracts sensitive notes (e.g., upgrade timelines, admin logic, emergency mechanisms).
4. Uses this to plan targeted attacks or front-run privileged operations.

**Assumptions:**

- Source code is published to Etherscan or included in verification metadata.
- Comments include confidential or overly descriptive information.
- Attackers are scanning public repositories and source code databases.

## ✅ Fixed Code

```solidity

// Basic comment indicating function purpose
function transfer(address to, uint256 value) public {
    // Transfers `value` tokens to `to`
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "No direct risk unless comments leak privileged or sensitive logic."
- context: "Contract with on-chain upgrade planning notes"
  severity: M
  reasoning: "Could provide adversaries with timing and strategy to exploit."
- context: "Minimal contract published with stripped source"
  severity: G
  reasoning: "No functional or information disclosure risk present."
```

## 🛡️ Prevention

### Primary Defenses

- Use NatSpec for interface documentation only.
- Avoid including operational notes, upgrade plans, or secret key references in comments.
- Strip non-NatSpec comments before contract verification.

### Additional Safeguards

- Maintain sensitive notes in off-chain secure documentation tools.
- Separate development and production branches with pre-deployment comment checks.

### Detection Methods

- Slither comments-too-long heuristic or custom plugin
- Manual Review Contract source audit before verification
- Bytecode Inspection Check for excessive metadata or inflated size

## 🕰️ Historical Exploits

- **Name:** Minswap Vulnerability Disclosure  
- **Date:** 2022  
- **Loss:** $0 – No financial loss  
- **Post-mortem:** [Link to post-mortem](https://www.reddit.com/r/cardano/comments/tn9edi/minswap_vulnerability_postmortem_must_read_to/)

## 📚 Further Reading

- [Solidity Documentation: NatSpec Comments](https://docs.soliditylang.org/en/latest/natspec-format.html)    
- [Ethereum Smart Contract Security Best Practices](https://ethereum.org/en/developers/docs/smart-contracts/security/)  

## ✅ Vulnerability Report

```markdown
id: LS09I
title: Excessive Comments 
severity: I
score:
impact: 1       
exploitability: 1 
reachability: 2 
complexity: 1     
detectability: 5  
finalScore: 1.45
```

---

## 📄 Justifications & Analysis

- **Impact**: Unless comments leak sensitive ops details, impact is minimal.
- **Exploitability**: Attackers must parse verified source or scraped metadata.
- **Reachability**: Depends on verification and whether source is public.
- **Complexity**: No execution required; it’s a passive information leak.
- **Detectability**: Easily caught via linting tools or Slither plugins.

