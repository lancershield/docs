# Inflation Amplification

```YAML
id: LS40H
title: Inflation Amplification 
baseSeverity: H
category: tokenomics
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: Token devaluation, supply runaway, protocol instability
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-704
swc: SWC-136
``` 

## 📝 Description

- Inflation amplification refers to a token minting mechanism that inadvertently multiplies total supply over time due to:
- Recursive rewards or rebasing logic
- Loops that distribute rewards based on previously inflated balances
- Failure to cap emissions or align them with real economic inputs
- This flaw is common in staking contracts, rebasing tokens, or liquidity mining systems where rewards are proportional to token balances, but the reward source is not capped or diluted after distribution.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract AmplifiedStaking {
    mapping(address => uint256) public balances;
    uint256 public totalSupply;

    function stake(uint256 amount) external {
        balances[msg.sender] += amount;
        totalSupply += amount;
    }

    function distributeRewards(uint256 rewardAmount) external {
        for (uint256 i = 0; i < 100; i++) {
            balances[msg.sender] += balances[msg.sender] * rewardAmount / totalSupply; // ❌ amplifies over time
        }
        totalSupply += rewardAmount; // ❌ adds total inflation on top of recursive increase
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A developer writes a distributeRewards() function that rewards users proportionally to their current balance.
2. A user deposits a large amount of tokens early and becomes a dominant share holder in the pool.
3. The contract mints new rewards without capping or normalizing them — and directly adds them to user balances.
4. In the next reward cycle, the user’s already-rewarded balance is used again to compute even more rewards, leading to exponential compounding.

**Assumptions:**

- Emissions are based on previously rewarded balances
- There's no supply cap or emission decay
- The loop fails to normalize rewards over time

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract CappedStaking {
    mapping(address => uint256) public balances;
    mapping(address => uint256) public rewards;
    uint256 public totalSupply;
    uint256 public totalRewardPerTokenStored;
    uint256 public rewardCap = 1_000_000 ether;

    function stake(uint256 amount) external {
        balances[msg.sender] += amount;
        totalSupply += amount;
    }

    function distributeRewards(uint256 rewardAmount) external {
        require(totalSupply > 0, "No stakers");
        require(rewardAmount <= rewardCap, "Reward exceeds cap");
        totalRewardPerTokenStored += rewardAmount * 1e18 / totalSupply;
        rewardCap -= rewardAmount;
    }

    function claim() external {
        uint256 reward = balances[msg.sender] * totalRewardPerTokenStored / 1e18 - rewards[msg.sender];
        rewards[msg.sender] += reward;
        // Transfer reward (omitted)
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Rewards minted in real tokens with economic value"
  severity: H
  reasoning: "Unbounded inflation causes economic damage and ecosystem risk"
- context: "Testnet or cosmetic reward logic only"
  severity: L
  reasoning: "No real loss, only display issue"
- context: "Rewards use pull-based capped emission model"
  severity: I
  reasoning: "Fully mitigated with enforced caps and vesting"
```

## 🛡️ Prevention

### Primary Defenses

- Use rewardPerToken models that track normalized rewards
- Never compound staking balances during distribution
- Cap emissions per epoch or with decay curves

### Additional Safeguards

- Monitor total supply growth with invariant checks
- Simulate reward curves under extreme inputs
- Use governance to pause or reduce emissions when runaway inflation occurs

### Detection Methods

- Check for reward logic that multiplies current balance repeatedly
- Tools: Slither (incorrect-math, suspicious-arithmetic), invariant testing, fuzzing

## 🕰️ Historical Exploits
 
- **Name:** Basis Cash Rebase Loop 
- **Date:** 2020-12 
- **Loss:** ~100% value drop due to compounded rebasing with no demand peg 
- **Post-mortem:** [Link to post-mortem](https://basis.cash/) 
  
## 📚 Further Reading

- [SWC-136: Unrestricted Critical Variable Update](https://swcregistry.io/docs/SWC-136)
- [CWE-704: Incorrect Type Conversion or Cast](https://cwe.mitre.org/data/definitions/704.html) 
- [OpenZeppelin – Token Emission Best Practices](https://docs.openzeppelin.com/contracts/4.x/tokens#emission-controls) 

---

## ✅ Vulnerability Report

```markdown
id: LS40H
title: Inflation Amplification 
severity: H
score:
impact: 5      
exploitability: 3 
reachability: 4  
complexity: 3     
detectability: 3  
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: System-wide failure via hyperinflation, loss of trust, and TVL drain
- **Exploitability**: Can be abused by whales to compound rewards disproportionately
- **Reachability**: Seen in protocols with staking, bonding, and rebasing logic
- **Complexity**: Rooted in economic math, not just Solidity logic
- **Detectability**: Requires testnets, simulations, or expert review of incentive mechanisms
