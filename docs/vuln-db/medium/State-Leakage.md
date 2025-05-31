# State Leakage 

```YAML
id: LS19M
title: State Leakage 
baseSeverity: M
category: information-disclosure
language: solidity
blockchain: [ethereum]
impact: Sensitive state data visible to anyone via public view calls
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-200
swc: SWC-136
```

## 📝 Description

- In Solidity, public state variables are automatically assigned a getter function, which exposes the underlying data on-chain and off-chain. 
- If developers mark sensitive mappings, arrays, or structs as public, they unintentionally reveal:
- Private financial positions
- Referral structures or user roles
- Whitelists/blacklists
- related metadata
- Because public mappings and arrays allow per-index queries, even partial exposure can enable systematic enumeration, profiling, and targeted exploitation.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract ReferralSystem {
    mapping(address => address) public referrer; // ❌ reveals user relationship
    mapping(address => uint256) public claimedAmounts; // ❌ exposes earnings

    function claimReward() external {
        // internal logic using claimedAmounts and referrer
    }
}
```

## 🧪 Exploit Scenario

1. A dApp uses a public mapping referrer[address] to track user affiliations.
2. A competitor queries this mapping across known addresses to scrape the referral graph.
3. They run a social engineering campaign against top referrers or impersonate them on alternative chains.
4. Alternatively, they monitor claimedAmounts() to profile profitable users and deploy targeted phishing or incentive campaigns.

**Assumptions:**

- public keyword is used on internal or sensitive mappings.
- Project assumes "view" functions are harmless or invisible (they are not).

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SecureReferralSystem {
    mapping(address => address) private _referrer;
    mapping(address => uint256) private _claimedAmounts;

    function getMyReferrer() external view returns (address) {
        return _referrer[msg.sender]; // ✅ returns only caller's own data
    }

    function getMyClaimedAmount() external view returns (uint256) {
        return _claimedAmounts[msg.sender];
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Common mistake in multi-stage logic; impact limited to session."
- context: "Upgradeable contract using shared storage"
  severity: H
  reasoning: "Shared variables may lead to critical cross-function logic errors."
- context: "Access-controlled payout with tight validation"
  severity: L
  reasoning: "Low-risk if contract performs full checks and resets state."
```

## 🛡️ Prevention

### Primary Defenses

- Never declare mappings, arrays, or structs as public if they contain sensitive data.
- Use private and define explicit getter functions with access control.

### Additional Safeguards

- Implement access roles for data views using Ownable, AccessControl, or similar.
- Consider obfuscating off-chain views for analytics via subgraphs, not contracts.

### Detection Methods

- Scan for public mapping(...) or public struct ....
- Review storage declarations that imply relationships or financial state.
- Tools: Slither (public-data-leak), manual audit, Semgrep

## 🕰️ Historical Exploits

- **Name:** Vulseye Discovery of State Leakage Vulnerabilities 
- **Date:** 2024 
- **Loss:** 42,738 smart contracts
- **Post-mortem:** [Link to post-mortem](https://arxiv.org/html/2408.10116v1)
  
## 📚 Further Reading

- [SWC-136: Unobservable or Improperly Observed Behavior](https://swcregistry.io/docs/SWC-136/) 
- [Solidity Docs – Variable Visibility](https://docs.soliditylang.org/en/latest/contracts.html#visibility-and-getters) 
- [OpenZeppelin Contracts – AccessControl](https://docs.openzeppelin.com/contracts/4.x/access-control) 
- [Slither – Public Data Leak Detector](https://github.com/crytic/slither/wiki/Detector-Documentation#public-state-variable)

---

## ✅ Vulnerability Report

```markdown
id: LS19M
title: State Leakage 
severity: M 
score:
impact: 3 
exploitability: 2 
reachability: 4 
complexity: 1   
detectability: 5 
finalScore: 3.0
```

---

## 📄 Justifications & Analysis

- **Impact**: User data and relationships are exposed on-chain and can be profiled.
- **Exploitability**: Anyone can scan public mappings and access historical states.
- **Reachability**: Common in staking, referral, whitelist, and claim systems.
- **Complexity**: Very low; caused by use of public visibility.
- **Detectability**: Easy to catch with Slither or even a visual code scan.