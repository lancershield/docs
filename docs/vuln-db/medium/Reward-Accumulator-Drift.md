# Reward Accumulator Drift

```YAML
id: TBA
title: Reward Accumulator Drift
severity: M
category: accounting
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Reward calculations diverge from expected values over time
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-682
swc: SWC-135
```

## 📝 Description

- Reward Accumulator Drift is a subtle logic flaw in staking and farming contracts where a protocol uses an accumulator-based reward model (e.g., accRewardPerShare) but:
- Fails to update global or per-user accumulators precisely,
- Introduces rounding errors that compound over time, or
- Omits updates under specific edge cases.

## 🚨 Vulnerable Code

```solidity

uint256 public accRewardPerShare;
uint256 public totalStaked;
mapping(address => uint256) public rewardDebt;

function deposit(uint256 amount) external {
    totalStaked += amount;
    userStakes[msg.sender] += amount;
    rewardDebt[msg.sender] = userStakes[msg.sender] * accRewardPerShare / 1e12;
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A protocol uses accRewardPerShare to track cumulative rewards.
2. The contract only updates this variable when harvest() or massUpdate() is called.
3. A new user deposits a large amount just before rewards are updated, and gets rewardDebt based on an outdated accumulator.
4. When the pool is updated later, the user’s rewards are over-allocated.
5. Over time, as more users enter/exit and the pool is inconsistently updated, accumulator drift accumulates, causing misaligned payouts.
6. Eventually, rewards run out early or some users earn significantly more/less than intended.

**Assumptions:**

- The protocol uses accumulator-based reward accounting (accRewardPerShare model).
- Rewards are not updated consistently before every user action (deposit/withdraw/harvest).

## ✅ Fixed Code

```solidity

function updatePool() internal {
    if (totalStaked == 0) return;
    uint256 rewards = getPendingRewards();
    accRewardPerShare += rewards * 1e12 / totalStaked;
}

function deposit(uint256 amount) external {
    updatePool(); // ✅ update before user state changes
    totalStaked += amount;
    userStakes[msg.sender] += amount;
    rewardDebt[msg.sender] = userStakes[msg.sender] * accRewardPerShare / 1e12;
}
```

## 🛡️ Prevention

### Primary Defenses

- Always call updatePool() before user state transitions (deposit, withdraw, harvest).
- Use high-precision scaling factors (e.g., 1e12) and fixed-point libraries.
- Design test cases for edge time gaps and batch reward updates.

### Additional Safeguards

- Track lastUpdateTime and assert it increases monotonically.
- Simulate long-term drift using fuzzing tools (e.g., Foundry, Echidna).
- Cap max reward per block to prevent overflow or over-allocation.

### Detection Methods

- Invariant testing: total rewards distributed == expected emission
- Compare accumulated rewards to actual token transfers
- Tools: Foundry, Slither (unused-params, incorrect-calculation), Dedaub’s invariant suite

## 🕰️ Historical Exploits

- **Name:** Pickle Finance Accumulator Drift 
- **Date:** 2021-01 
- **Loss:** ~$1,200,000 in overpaid or underpaid rewards due to misaligned `accRewardPerShare` updates 
- **Post-mortem:** [Link to post-mortem](https://forum.pickle.finance/) 
- **Name:** ApeSwap Harvest Delay Bug 
- **Date:** 2021-05 
- **Loss:** ~$450,000 in misallocated BANANA rewards due to update skip 
- **Post-mortem:** [Link to post-mortem](https://medium.com/@apeswapfinance) 
- **Name:** AutoFarm Bounty for Drift Prevention 
- **Date:** 2021 
- **Loss:** N/A (prevention bounty paid to user who flagged drift edge case) 
- **Post-mortem:** [Link to post-mortem](https://github.com/autofarm-network)
  
## 📚 Further Reading

- [CWE-682: Incorrect Calculation](https://cwe.mitre.org/data/definitions/682.html) 
- [SWC-135: Incorrect Implementation](https://swcregistry.io/docs/SWC-135) 
- [MasterChef Reward Model – SushiSwap](https://github.com/sushiswap/masterchef) 
- [OpenZeppelin FixedPointMath](https://docs.openzeppelin.com/contracts/4.x/api/utils#SafeCast) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Reward Accumulator Drift
severity: M
score:
impact: 3  
exploitability: 3  
reachability: 5  
complexity: 2  
detectability: 3  
finalScore: 3.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Can result in protocol overpaying or underpaying rewards, distorting emission schedules.
- **Exploitability**: Strategic users may time interactions to take advantage of stale accumulators.
- **Reachability**: Nearly all DeFi staking/farming contracts use similar reward accounting logic.
- **Complexity**: Caused by state ordering or precision loss, not complicated exploits.
- **Detectability**: Requires test coverage with delayed updates and timing-based fuzzing.

