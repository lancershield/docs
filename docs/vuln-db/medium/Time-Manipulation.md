# Time Manipulation vulnerability

```YAML
id: TBA
title: Time Manipulation Vulnerabilities 
severity: M
category: time-manipulation
language: solidity
blockchain: [ethereum]
impact: Early/late access to functions, payout manipulation, or DoS
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-829
swc: SWC-116
``` 

## 📝 Description

- Time manipulation vulnerabilities occur when a contract relies on `block.timestamp` (or `now` in older Solidity) for critical logic such as access control, randomness, or scheduled operations. 
- Since miners have **limited but real control** over block timestamps (±15 seconds), they can manipulate execution order, bypass time locks, or gain advantage in games and staking systems. Over-reliance on `block.timestamp` without safeguards results in **unexpected execution or unfair conditions**.

## 🚨 Vulnerable Code

```solidity
contract TimeLock {
    uint256 public unlockTime;

    function lockTokens(uint256 duration) external {
        unlockTime = block.timestamp + duration;
    }

    function withdraw() external {
        require(block.timestamp >= unlockTime, "Tokens still locked");
        // transfer tokens
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A staking contract uses block.timestamp to set unlockTime.
2. An attacker runs a node (or colludes with a miner) and mines a block with a slightly increased timestamp.
3. This causes withdraw() to be callable earlier than intended.
4. The attacker can front-run normal users and withdraw tokens unfairly.

**Assumptions:**

- Miner has influence on timestamp within consensus rules.
- Timestamp determines access to funds, rewards, or actions.

## ✅ Fixed Code

``` solidity

contract TimeLock {
    uint256 public unlockBlock;

    function lockTokens(uint256 numBlocks) external {
        unlockBlock = block.number + numBlocks;
    }

    function withdraw() external {
        require(block.number >= unlockBlock, "Tokens still locked");
        // transfer tokens
    }
}
``` 

## 🛡️ Prevention

### Primary Defenses

- Use block.number instead of block.timestamp for measuring time duration.
- Only use block.timestamp when exact timing is not critical.

### Additional Safeguards

- Add buffer windows (e.g., require block.timestamp > t + Δ) for safety.
- If using timestamp, never use it for randomness or single-trigger logic.
- Use Chainlink Keepers or off-chain cron services for precise scheduling.

### Detection Methods

- Slither: timestamp-dependence detector.
- Manual audit of logic involving block.timestamp, especially in require() or access modifiers.
- Simulation of off-bound timestamps via fuzzing or hardhat tests.

## 🕰️ Historical Exploits

- **Name:** Rubixi Ponzi Scheme
- **Date:** 2016 
- **Loss:** Unspecified 
- **Post-mortem:** [Link to post-mortem](https://www.sciencedirect.com/science/article/pii/S1546221824002285)
  
## 📚 Further Reading

- [SWC-116: Timestamp Dependence](https://swcregistry.io/docs/SWC-116) 
- [Timestamp Dependency in Smart Contracts – GeeksforGeeks](https://www.geeksforgeeks.org/timestamp-dependency-in-smart-contracts/) 
- [Smart Contract Vulnerability Dataset – GitHub](https://github.com/imranpollob/smart-contract-vulnerability-dataset) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Time Manipulation Vulnerabilities 
severity: M
score:
impact: 3         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Minor miner manipulation may lead to early/late unlocks or protocol desyncs.
- **Exploitability**: Miners or transaction bundlers can easily exploit short windows.
- **Reachability**: Time-based conditions are frequent in staking, auctions, and lottery logic.
- **Complexity**: Simple attack if you control the block, especially in low-hashrate networks.
- **Detectability**: Timestamp dependence is easily flagged in audits and static analysis tools.
