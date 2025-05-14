# Reward Inflation Bugs

```YAML
id: TBA
title: Reward Inflation Bugs from Miscalculated or Unbounded Emissions
severity: H
category: tokenomics
language: solidity
blockchain: [ethereum]
impact: Excessive or infinite reward generation causing economic collapse
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-682
swc: SWC-135
```

## 📝 Description

- Reward inflation bugs occur when token rewards (for staking, farming, or liquidity provision) are over-allocated, unbounded, or miscalculated due to logic flaws or missing constraints. 
- This leads to:
- Exponential or incorrect reward issuance,
- Devaluation of the native token,
- Potential collapse of the economic model or drain of reward pools.
- Often the cause lies in improperly updating `accRewardPerShare`, incorrect time tracking, or failing to cap multipliers and emissions.

## 🚨 Vulnerable Code

```solidity
contract InflationBug {
    mapping(address => uint256) public staked;
    mapping(address => uint256) public rewardDebt;
    uint256 public accRewardPerShare;
    uint256 public lastRewardTime;
    uint256 public rewardRate = 10 ether; // per second

    function updatePool() public {
        uint256 timeElapsed = block.timestamp - lastRewardTime;
        accRewardPerShare += timeElapsed * rewardRate; // ❌ no scaling or bounds
        lastRewardTime = block.timestamp;
    }

    function pendingReward(address user) public view returns (uint256) {
        return staked[user] * accRewardPerShare - rewardDebt[user];
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Contract has rewardRate = 10 ether, no upper limit, and is idle for 10 days.
2. A user stakes right before updatePool() is called.
3. timeElapsed is large, resulting in massive inflation of accRewardPerShare.
4. The user claims rewards worth thousands of tokens for a minimal stake.
5. Reward pool is drained, and token price crashes due to excess emission.

**Assumptions:**

- No upper bound or time capping logic in reward multiplier.
- accRewardPerShare is not scaled or protected from overflow.
- Pool can sit idle for long periods.

## ✅ Fixed Code

```solidity

function updatePool() public {
    uint256 currentTime = block.timestamp;
    if (currentTime > lastRewardTime) {
        uint256 timeElapsed = currentTime - lastRewardTime;

        // ✅ Cap the max multiplier to prevent excessive reward jumps
        if (timeElapsed > 86400) {
            timeElapsed = 86400; // max 1 day
        }

        accRewardPerShare += (timeElapsed * rewardRate) / 1e18;
        lastRewardTime = currentTime;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always scale accRewardPerShare to prevent overflows (e.g., divide by 1e18).
- Cap time-based multipliers to prevent abuse from idle pools.
- Use safeRewardRate, maxEmissionRate, or MAX_DURATION constants.

### Additional Safeguards

- Emit UpdatePool logs with timestamps for off-chain tracking.
- Audit the reward formula for linearity and expected inflation under edge cases.
- Use automated tests to simulate long idle durations and reward cliffs.

### Detection Methods

- Slither: custom detectors for accRewardPerShare growth and unchecked multipliers.
- Formal modeling or fuzzing reward logic under long time intervals.
- Unit test suites simulating multi-day inactivity and claim bursts.

## 🕰️ Historical Exploits

- **Name:** Pickle Finance Inflation Miscalculation 
- **Date:** 2020 
- **Loss:** Misallocation of yield rewards due to scaling error 
- **Post-mortem:** [Link to post-mortem](https://medium.com/pickle-finance/pickle-finance-post-mortem-74d5a4b4e489) 


## 📚 Further Reading

- [SWC-135: Code With No Effects](https://swcregistry.io/docs/SWC-135) 
- [Solidity Docs – Integer Math and Scaling](https://docs.soliditylang.org/en/latest/) 
- [Delphi – How to Design Safe Staking Rewards](https://www.delphidigital.io/reports/) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Reward Inflation Bugs f
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 3  
finalScore: 4.2

```

---

## 📄 Justifications & Analysis

- **Impact**: Can destroy protocol tokenomics by overpaying users or draining reward pools.
- **Exploitability**: Easily triggered after periods of low usage or deployment delay.
- **Reachability**: Appears in nearly all staking, farming, or yield protocols.
- **Complexity**: Moderate — attacker just needs timing or minimal capital.
- **Detectability**: Static tools may miss it; simulation or test coverage often required.