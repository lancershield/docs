# Insecure RewardDistributor

```YAML
id: TBA
title: Insecure RewardDistributor 
baseSeverity: H
category: reward-accounting
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: Reward theft, dilution of honest users, or reward pool draining
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-703
swc: SWC-136
```

## 📝 Description

- Reward distribution contracts often allow users to claim rewards based on stake duration, balance, or share of a global pool. If implemented incorrectly, they are vulnerable to:
- Overclaiming rewards by repeatedly calling claim()
- Reward dilution, where late participants receive a share of earlier allocations
- Skipped or missed updates to global reward state before per-user accounting
- Zero-share reward draining, where inactive accounts can claim rewards
- Most issues stem from the contract not updating reward state before allowing user interactions, or from failing to checkpoint user state before updates.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract RewardDistributor {
    mapping(address => uint256) public userRewardDebt;
    mapping(address => uint256) public balances;
    uint256 public rewardPerTokenStored;
    uint256 public totalSupply;

    function updateRewards(uint256 rewardAmount) external {
        rewardPerTokenStored += rewardAmount / totalSupply; // ❌ division before validation
    }

    function claim() external {
        uint256 owed = balances[msg.sender] * rewardPerTokenStored - userRewardDebt[msg.sender];
        userRewardDebt[msg.sender] += owed; // ❌ debt updated after transfer
        payable(msg.sender).transfer(owed); // ❌ no reentrancy guard
    }
}
```

## 🧪 Exploit Scenario

1. A malicious user deposits tokens and becomes 100% of totalSupply.
2. They call updateRewards(1000) and then claim() to receive all 1000 tokens.
3. Another user enters the pool after rewards are updated, with no fair share adjustment.
4. Or the attacker reenters claim() during the transfer, receiving rewards multiple times before userRewardDebt is updated.

**Assumptions:**

- No checkpointing of reward state on user entry/exit
- Global reward logic does not sync with user share accounting before critical ops

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SafeRewardDistributor is ReentrancyGuard {
    mapping(address => uint256) public userRewardDebt;
    mapping(address => uint256) public balances;
    uint256 public rewardPerTokenStored;
    uint256 public totalSupply;

    function updateRewards(uint256 rewardAmount) external {
        require(totalSupply > 0, "No stakers");
        rewardPerTokenStored += rewardAmount / totalSupply;
    }

    function claim() external nonReentrant {
        uint256 earned = balances[msg.sender] * rewardPerTokenStored - userRewardDebt[msg.sender];
        require(earned > 0, "Nothing to claim");
        userRewardDebt[msg.sender] += earned;
        payable(msg.sender).transfer(earned);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Live DeFi protocol using token rewards without state-reset logic"
  severity: H
  reasoning: "Attackers can drain entire reward pool through repeated claims"
- context: "Claim system protected via Merkle proofs or one-time withdrawal"
  severity: M
  reasoning: "Impact limited; exploit prevented through proof verification"
- context: "Testing/staging environment or limited distribution scope"
  severity: L
  reasoning: "Impact contained and recoverable"
```
## 🛡️ Prevention

### Primary Defenses

- Use OpenZeppelin’s nonReentrant on all claim-like functions
- Always checkpoint before transferring tokens
- Require non-zero totalSupply before calculating per-token rewards

### Additional Safeguards

- Store accRewardPerShare in scaled precision (e.g., *1e12) to reduce rounding loss
- Avoid mutable globals shared between uncheckpointed users

### Detection Methods

- Look for reward logic that fails to checkpoint before transfer
- Detect division on potentially-zero values
- Tools: Slither (divide-before-zero-check, reentrancy-vulnerability), MythX, audit

## 🕰️ Historical Exploits

- **Name:** Synthetix Inflation Reward Bug 
- **Date:** 2020-07 
- **Loss:** ~$50,000 
- **Post-mortem:** [Link to post-mortem](https://blog.synthetix.io) 
   
## 📚 Further Reading

- [SWC-136: Unrestricted Critical Variable Update](https://swcregistry.io/docs/SWC-136/) 
- [OpenZeppelin – ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) 
- [Synthetix Staking Architecture](https://docs.synthetix.io)
- [Solidity Patterns – Pull-over-Push](https://fravoll.github.io/solidity-patterns/pull_over_push.html)

---
 
## ✅ Vulnerability Report

```markdown
id: TBA
title: Insecure RewardDistributor 
severity: H
score:
impact: 4   
exploitability: 4 
reachability: 4  
complexity: 2    
detectability: 4 
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Rewards may be drained or incorrectly distributed
- **Exploitability**: Anyone can exploit with crafted input + timing
- **Reachability**: Found in nearly every DeFi reward contract
- **Complexity**: Slightly technical but widespread
- **Detectability**: Easily found via logic review or fuzzing