# Incentive Misalignment in Staking

```YAML
id: TBA
title: Incentive Misalignment in Staking
baseSeverity: M
category: economic
language: solidity
blockchain: [ethereum, polygon, arbitrum, optimism, bsc]
impact: Rational users exploit loopholes to maximize rewards unfairly
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-697: Incorrect Comparison
swc: SWC-131: Presence of unused or misused logic
```

## 📝 Description

- Incentive misalignment in staking occurs when rational users are able to optimize their behavior to gain more than their fair share of rewards, often by:
- Looping deposits and withdrawals around snapshot intervals
- Front-running reward updates
- Creating multiple wallets to farm “per-user” incentives
- Such economic loopholes result not from code bugs but from flawed reward design logic, where the protocol inadvertently rewards behavior that undermines its goals—such as short-term staking being more profitable than long-term holding.
- This is often a result of:
- Reward calculations that rely on balance at time T rather than duration-weighted stake
- Failing to penalize early withdrawal or lazy staking
- Using per-user reward pools without controls

## 🚨 Vulnerable Code

```solidity

function claimReward() external {
    uint256 reward = stakingBalance[msg.sender] * rewardRate;
    rewardToken.transfer(msg.sender, reward);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Staking pool calculates rewards using current balance without considering stake duration.
2. A user monitors the reward cycle and deposits a large amount shortly before claimReward() is triggered.
3. The user receives a large share of the reward pool despite not contributing meaningfully over time.
4. After claiming, the user withdraws immediately and repeats the loop in the next cycle.
5. Genuine long-term stakers earn significantly less due to dilution.

**Assumptions:**

- Reward distribution is snapshot-based or balance-based without time weighting.
- No cooldowns, lockups, or penalty mechanics are in place.
- Users can repeatedly deposit/withdraw within a reward cycle.

## ✅ Fixed Code

```solidity

struct StakeInfo {
    uint256 amount;
    uint256 timestamp;
}

mapping(address => StakeInfo) public stakes;

function claimReward() external {
    uint256 stakedTime = block.timestamp - stakes[msg.sender].timestamp;
    uint256 reward = stakes[msg.sender].amount * stakedTime * rewardRate;
    stakes[msg.sender].timestamp = block.timestamp;
    rewardToken.transfer(msg.sender, reward);
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Reward imbalance can reduce participation and allow reward sniping."
- context: "High-value DeFi staking protocol"
  severity: H
  reasoning: "Can lead to centralization or drain of protocol rewards."
- context: "Private staking with off-chain distribution"
  severity: L
  reasoning: "Protocol controls rewards manually, less risk of abuse."
```

## 🛡️ Prevention

### Primary Defenses

- Use time-weighted staking (e.g., stake amount × duration).
- Implement cooldown periods before reward eligibility.
- Apply reward multipliers for longer commitments or vesting curves.

### Additional Safeguards

- Introduce slashing or early exit penalties.
- Enforce snapshot-based locking before reward distribution windows.
- Rate-limit reward claims per user or per epoch.

### Detection Methods

- Simulate economic scenarios with adversarial actors.
- Use agent-based modeling or Foundry fuzzing to identify edge behavior.
- Review if APY logic incentivizes minimal-stake-max-claim patterns.

## 🕰️ Historical Exploits

- **Name:** Lido stETH Reward Skimming 
- **Date:** 2022-07 
- **Loss:** N/A 
- **Post-mortem:** [Link to post-mortem](https://github.com/lidofinance) 
  
## 📚 Further Reading

- [SWC-131: Presence of Unused or Misused Logic](https://swcregistry.io/docs/SWC-131) 
- [CWE-697: Incorrect Comparison](https://cwe.mitre.org/data/definitions/697.html)   
- [Delphi Digital – Token Incentive Design](https://www.delphidigital.io/) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Incentive Misalignment in Staking
severity: M
score:
impact: 3  
exploitability: 4  
reachability: 5 
complexity: 2   
detectability: 3  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Rational actors gain excess rewards, reducing long-term user trust and destabilizing token incentives.
- **Exploitability**: Easily triggered via off-chain analysis and on-chain transactions timed around reward events.
- **Reachability**: Common in simplistic staking implementations or per-block reward logic.
- **Complexity**: Requires minimal tooling—timing and script automation suffice.
- **Detectability**: Detectable through scenario simulation, but often missed in standard static audits.


