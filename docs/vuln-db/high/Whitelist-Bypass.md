# Whitelist Bypass

```YAML
id: TBA
title: Whitelist Bypass 
severity: H
category: access-control
language: solidity
blockchain: [ethereum]
impact: Unauthorized access to restricted features
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-284
swc: SWC-105
```

## 📝 Description

- A Whitelist Bypass occurs when a contract's access control checks meant to restrict functionality to approved addresses (e.g., for presales, governance participation, or privileged calls) are either absent, implemented incorrectly, or easily circumvented. 
- These logic flaws allow non-whitelisted users to perform sensitive operations reserved for approved users, potentially leading to loss of funds, ecosystem manipulation, or broken assumptions in tokenomics or access flows.

## 🚨 Vulnerable Code

```solidity
mapping(address => bool) public whitelist;

function participateInPresale() external {
    require(tx.origin == msg.sender, "No contracts allowed");
    if (!whitelist[msg.sender]) {
        // warning only — no revert!
        emit NotWhitelisted(msg.sender);
    }
    _processPurchase(msg.sender);
}
```

## 🧪 Exploit Scenario

1. Step-by-step exploit process:
2. A malicious user is not added to the whitelist.
3. They call participateInPresale() directly.
4. Since the check doesn’t revert or return, the function still processes their purchase.
5. The attacker bypasses whitelist restrictions and acquires tokens or access reserved for approved users.

**Assumptions:**

- Developer incorrectly assumes logging a failed access is sufficient.
- No secondary contract or frontend-level checks are enforced.
- Tokens, funds, or functionality are gated behind a flawed whitelist check.

## ✅ Fixed Code

```solidity

mapping(address => bool) public whitelist;

function participateInPresale() external {
    require(whitelist[msg.sender], "Not whitelisted");
    _processPurchase(msg.sender);
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Grants unauthorized users access to restricted functions, like token mints or governance votes."
- context: "Whitelist-based NFT presale"
  severity: H
  reasoning: "Attackers can mint rare NFTs unfairly or drain allocation."
- context: "Private governance settings"
  severity: M
  reasoning: "Impact depends on what functionality is behind the whitelist."
```

## 🛡️ Prevention

## Primary Defenses

- Always enforce require(whitelist[msg.sender]) or equivalent checks before protected logic.
- Do not rely on tx.origin, emitted events, or frontend logic for access control.

### Additional Safeguards

- Consider role-based access using AccessControl or Ownable.
- Separate user-facing functions from internal logic and restrict entrypoints.
- Use Merkle roots for scalable whitelist management and on-chain verification.

### Detection Methods

- Slither: access-control, missing-authorization detectors.
- Manual audit of functions that include allowlists or user-role mapping.
- Tests that simulate access attempts by non-whitelisted addresses.

## 🕰️ Historical Exploits

- **Name:** xToken Whitelist Bypass 
- **Date:** 2021-05-12 
- **Loss:** ~$24 million 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/xtoken-rekt/) 
  
## 📚 Further Reading

- [SWC-105: Unprotected Critical Function – SWC Registry](https://swcregistry.io/docs/SWC-105/) 
- [OpenZeppelin: Access Control Best Practices](https://docs.openzeppelin.com/contracts/4.x/access-control) 
- [Cyfrin CodeHawks - Whitelist Bypass Ambiguity](https://codehawks.cyfrin.io/c/2025-02-raac/s/1671)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Whitelist Bypass via Faulty Authorization Logic
severity: H
score:
impact: 4        
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Bypassed whitelist logic can undermine tokenomics, presales, or voting.
- **Exploitability**: Easily triggered due to flawed require logic or missing reverts.
- **Reachability**: Publicly accessible functions with access control expectations.
- **Complexity**: No on-chain infrastructure or simulation required.
- **Detectability**: High — basic review or tools like Slither will flag this.

