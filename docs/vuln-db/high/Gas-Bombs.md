# Gas Bombs

```YAML
id: TBA
title: Gas Bombs in Unbounded State Changes
baseSeverity: H
category: denial-of-service
language: solidity
blockchain: [ethereum]
impact: Out-of-gas failures and transaction censorship
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-400
swc: SWC-128
```

## 📝 Description

- A Gas Bomb vulnerability arises when a contract operation causes an unexpectedly high amount of gas consumption, making the function effectively unusable once triggered. This often results from:
- Unbounded loops over user-controlled arrays or mappings.
- Nested storage writes inside loops.
- Mass state mutations in a single transaction.
- An attacker can exploit this to create denial-of-service (DoS) conditions, prevent execution of vital contract logic, or break composability with other contracts.

## 🚨 Vulnerable Code

```solidity

mapping(address => uint256[]) public userTxs;

function deleteAllUserTxs(address user) external {
    for (uint256 i = 0; i < userTxs[user].length; i++) {
        delete userTxs[user][i];
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker creates hundreds or thousands of entries in userTxs[msg.sender].
2. Later, the attacker (or another user) calls deleteAllUserTxs().
3. The function runs out of gas during execution due to excessive loop iterations.
4. The contract function becomes effectively unusable and blocks further logic.

**Assumptions:**

- The array or mapping is attacker-controlled and unbounded.
- There is no gas cap, pagination, or batched processing.
- The logic is not protected against mass execution.

## ✅ Fixed Code

```solidity
function deleteUserTxsInRange(address user, uint256 start, uint256 end) external {
    require(end > start, "Invalid range");
    require(end <= userTxs[user].length, "Out of bounds");

    for (uint256 i = start; i < end; i++) {
        delete userTxs[user][i];
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Critical user withdrawals or admin-only actions blocked"
  severity: H
  reasoning: "Funds or state permanently inaccessible due to gas limits"
- context: "Rewards can be claimed via pull model instead of loop"
  severity: M
  reasoning: "Users can bypass central DoS but may suffer UX issues"
- context: "Gas-bounded batch logic with checkpoints in place"
  severity: L
  reasoning: "Low risk of gas exhaustion under normal use"
```

## 🛡️ Prevention

### Primary Defenses

- Always bound loop iterations using reasonable limits.
- Avoid modifying large storage arrays or mappings in a single call.
- Apply pagination or batched processing patterns to handle large datasets.

### Additional Safeguards

- Use gas estimation in tests for edge cases.
- Set maximum element thresholds for dynamic storage.
- Add circuit breakers to prevent overloaded execution paths.

### Detection Methods

- Slither: unbounded-loop, dos-with-unbounded-operation detectors.
- Mythril: symbolic gas analysis for infinite or heavy loops.
- Manual review of any function iterating over mappings/arrays.

## 🕰️ Historical Exploits

- **Name:** Boss Bridge Gas Bomb Exploit 
- **Date:** 2023 
- **Loss:** Denial-of-service due to excessive gas consumption 
- **Post-mortem:** [Link to post-mortem](https://updraft.cyfrin.io/courses/security/bridges/gas-bomb?lesson_format=video)
  

## 📚 Further Reading

- [SWC-128: DoS via Block Gas Limit – SWC Registry](https://swcregistry.io/docs/SWC-128/) 
- [Ethereum Stack Exchange: How much gas does it cost to emit an event?](https://ethereum.stackexchange.com/questions/106772/how-much-gas-does-it-cost-to-emit-an-event) 
- [Smart Contract Gas Optimization Techniques – Unvest](https://www.unvest.io/blog/smart-contract-gas-optimization-techniques-how-to-write-efficient-contracts-to-minimize-gas-fees)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Gas Bombs in Unbounded State Changes
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2    
detectability: 3  
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Target function is effectively disabled due to gas limits.
- **Exploitability**: Publicly exposed unbounded state changes are trivial to exploit.
- **Reachability**: Present in many storage-cleaning or bulk-processing functions.
- **Complexity**: Medium complexity; attacker needs to inflate state then call a loop.
- **Detectability**: Detectable by Slither and Mythril, but sometimes overlooked in manual review.

