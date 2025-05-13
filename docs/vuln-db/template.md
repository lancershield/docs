# Title of the vulnerability

```YAML
id: TBA  # Leave this as "TBA"; the team will assign the official ID
title: Example Vulnerability Title
severity: M # Options: C | H | M | L | I | G
category: access-control
language: solidity
blockchain: [ethereum]
impact: Unauthorized asset withdrawal
status: draft # Options: draft | verified | published
complexity: low # Options: low | medium | high
attack_vector: external
mitigation_difficulty: easy # Options: easy | medium | hard
versions: [">0.6.0", "<0.8.0"]
cwe: CWE-284
swc: SWC-105
```

## 📝 Description

Explain the vulnerability in clear terms. Focus on:

- What fundamental flaw creates this vulnerability
- Where this pattern commonly appears in smart contracts
- Why this vulnerability is dangerous (technical impact)

## 🚨 Vulnerable Code

```solidity
// Replace with a minimal, complete example
function withdraw(uint amount) public {
    require(balances[msg.sender] >= amount);
    (bool sent, ) = msg.sender.call{value: amount}("");
    require(sent);
    balances[msg.sender] -= amount;
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker deploys a malicious contract that...
2. The attacker then calls...
3. During execution, the vulnerable contract...
4. This allows the attacker to...

**Assumptions:** List any prerequisites for the attack

## ✅ Fixed Code

```solidity
// Safer implementation
function withdraw(uint amount) public {
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount;
    (bool sent, ) = msg.sender.call{value: amount}("");
    require(sent);
}
```

## 🛡️ Prevention

### Primary Defenses

- Most important prevention technique
- Second most important technique

### Additional Safeguards

- Additional defensive measures
- Testing approaches

### Detection Methods

- How to detect this issue in existing code
- Tools that can help identify this vulnerability

## 🕰️ Historical Exploits

Case studies, notable incidents, or real-world examples relevant to this vulnerability.

<!--
Example:
- **Name:** MyDefiVault Hack
  **Date:** 2021-04-15
  **Loss:** $1.2M
  **Post-mortem:** [Link to post-mortem](https://example.com/post-mortem)
-->

## 📚 Further Reading

Additional resources, tools, or discussions related to this vulnerability.

<!--
Example:
- [SWC Registry: Authorization Through tx.origin](https://swcregistry.io/docs/SWC-105)
- [OpenZeppelin: Access Control Best Practices](https://example.com/post-mortem)
-->

---

## ✅ Vulnerability Report Template

```markdown
id: <unique-vulnerability-id>
title: <vulnerability-title>
severity: < H | M | L | I | G>
score:
impact: <0-5>
exploitability: <0-5>
reachability: <0-5>
complexity: <0-5>
detectability: <0-5>
finalScore: <calculated-weighted-score>
```

---

## 📄 Justifications & Analysis

Provide technical rationales for each axis score here:

- **Impact**: [Explain why this bug would (or wouldn’t) cause financial/state loss]
- **Exploitability**: [Clarify the conditions under which this can be triggered]
- **Reachability**: [Is the code path realistically invoked? Any blockers?]
- **Complexity**: [How much attacker effort, knowledge, or setup is required?]
- **Detectability**: [Would this be caught in a standard audit pipeline?]
