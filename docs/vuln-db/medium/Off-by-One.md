# Off-by-One Errors 

```YAML
id: TBA
title: Off-by-One Errors 
baseSeverity: M
category: logic
language: solidity
blockchain: [ethereum]
impact: Skipped iteration, unintended access, or incorrect logic branching
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-193
swc: SWC-123
```

## 📝 Description

- Off-by-One Errors are logic bugs that occur when loop indices, array lengths, or conditional boundaries are incorrectly calculated—typically off by a single unit. In Solidity, this can cause:
- The first or last element of an array to be skipped,
- Loops to iterate too many or too few times,
- Incorrect boundary checks resulting in unintended access or logic branching.
- In smart contracts, such subtle issues can lead to missed reward distributions, invalid token claims, or access control bypass depending on how indexing or counting is handled.

## 🚨 Vulnerable Code

```solidity
contract RewardDistributor {
    address[] public users;

    function distributeRewards() external payable {
        for (uint256 i = 1; i < users.length; i++) {
            payable(users[i]).transfer(msg.value / users.length);
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Project adds 100 users to users[].
2. distributeRewards() is called to evenly distribute funds.
3. The first user (index 0) receives no reward due to incorrect loop start index.
4. Funds are either locked or misallocated, breaking protocol assumptions.

**Assumptions:**

- Developers assumed loop starts at index 1 rather than 0.
- There’s no correction mechanism or audit to catch the bug.
- Users expect equal treatment or critical logic depends on iteration completeness.

## ✅ Fixed Code

```solidity

function distributeRewards() external payable {
    for (uint256 i = 0; i < users.length; i++) {
        payable(users[i]).transfer(msg.value / users.length);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Could revert or behave unexpectedly on boundary conditions."
- context: "Financial loop (e.g., reward or debt payment)"
  severity: H
  reasoning: "Single index error may cause payment failure or asset mismanagement."
- context: "Internal-only loop over fixed array"
  severity: L
  reasoning: "Low risk as input and loop range are tightly controlled."
```

## 🛡️ Prevention

### Primary Defenses

- Always validate loop boundaries and conditional checks carefully.
- Write automated tests that include boundary conditions (first/last index).
- Review compiler warnings and logic diff coverage.

### Additional Safeguards

- Use OpenZeppelin data structures like EnumerableSet to abstract indexing.
- Implement fuzz testing or invariant checks around critical iteration paths.
- Avoid manually writing index math when safe libraries can be used.

### Detection Methods

- Slither: incorrect-index-access, loop-boundary-error detectors.
- Mythril: symbolic analysis for unreachable/partial state logic.
- Manual logic testing with edge-case coverage.

## 🕰️ Historical Exploits

- **Name:** Furucombo Misconfigured Combo Index 
- **Date:** 2021 
- **Loss:** Internal state inconsistency due to logic indexing bug 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/furucombo-rekt/) 
  
## 📚 Further Reading

- [SWC-123: Requirement Violation – SWC Registry](https://swcregistry.io/docs/SWC-123/) 
- [CWE-193: Off-by-One Error – MITRE](https://cwe.mitre.org/data/definitions/193.html) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Off-by-One Errors 
severity: M
score:
impact: 3         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 3.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Can break fairness, payment, or logic assumptions.
- **Exploitability**: Not always exploitable, but leads to logic flaws.
- **Reachability**: Found in reward loops, admin lists, and iterators.
- **Complexity**: A subtle bug introduced via miscounting.
- **Detectability**: Detectable via static tools or detailed logic coverage tests.


