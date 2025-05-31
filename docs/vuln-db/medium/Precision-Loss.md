# Precision Loss 

```YAML
id: LS21M
title: Precision Loss 
baseSeverity: M
category: arithmetic
language: solidity
blockchain: [ethereum, optimism, arbitrum, polygon, bsc]
impact: Inaccurate payouts, rounding errors, and fund misallocation
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-1339
swc: SWC-131
```

## 📝 Description

- Precision loss arises when smart contracts perform arithmetic operations with inadequate decimal scaling, especially during:
- Token exchange rate calculations
- Fee or interest rate computation
- Proportional reward or dividend splits
- Since Solidity does not support floating point arithmetic, all calculations must be done using scaled integers (e.g., amount * 1e18 / rate). Omitting proper scaling or truncating during intermediate steps can lead to:
- Incorrect fund distribution
- Rounding errors that accumulate over time
- Unfair outcomes in staking, yield farming, bonding, or liquidity incentives

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract RewardVault {
    mapping(address => uint256) public shares;
    uint256 public totalShares;
    uint256 public rewardPool;

    function claimReward(address user) external {
        uint256 userShare = shares[user];
        uint256 reward = rewardPool * userShare / totalShares; // ❌ precision loss for small shares
        // Transfer reward (omitted)
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A protocol has 1,000,000 total shares.
2. A new user has 1 share and is eligible for a portion of the reward.
3. The user’s share becomes permanently unrecoverable due to rounding truncation.

**Assumptions:**

- Dividing contributions across multiple addresses
- Timing exit/redemption when rounding errors benefit them

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract PreciseRewardVault {
    uint256 public constant PRECISION = 1e18;

    mapping(address => uint256) public shares;
    uint256 public totalShares;
    uint256 public rewardPool;

    function claimReward(address user) external {
        uint256 userShare = shares[user];
        uint256 scaledReward = rewardPool * userShare * PRECISION / totalShares;
        uint256 reward = scaledReward / PRECISION;
        // Transfer reward (omitted)
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Precision issues may cause unfairness or small loss without full breakage."
- context: "Public DeFi protocol with rewards, fees, or voting"
  severity: H
  reasoning: "Critical in staking, DAO voting, and fee splits—may lead to measurable financial loss."
- context: "Private utility contract with limited scope"
  severity: L
  reasoning: "Impact is minor, affecting only non-critical outcomes or simulations."
```
## 🛡️ Prevention

### Primary Defenses

- Use consistent PRECISION constants for fractional logic
- Always multiply before divide when calculating ratios
- Unit test edge cases (very small / very large ratios)

### Additional Safeguards

- Consider libraries like PRBMath or FixedPoint
- Emit warnings or cap minimum values when precision loss is unavoidable

### Detection Methods

- Static analysis for integer division without prior scaling
- Tools: Slither (divide-before-multiply, truncation), custom lint rules

## 🕰️ Historical Exploits

- **Name:** Synthetix SNX Rewards Distribution Drift 
- **Date:** 2019-10 
- **Loss:** ~$7,000 
- **Post-mortem:** [Link to post-mortem](https://sips.synthetix.io/sips/sip-12/)  
  
## 📚 Further Reading

- [SWC-131: Incorrect Calculation](https://swcregistry.io/docs/SWC-131) 
- [CWE-1339: Precision Errors](https://cwe.mitre.org/data/definitions/1339.html) 
- [Solidity Docs – Integer Division](https://docs.soliditylang.org/en/latest/control-structures.html#mathematical-and-bitwise-operations)
 
---

## ✅ Vulnerability Report

```markdown
id: LS21M
title: Precision Loss 
severity: M
score:
impact: 3      
exploitability: 3 
reachability: 5   
complexity: 2    
detectability: 5  
finalScore: 3.4
```

---

## 📄 Justifications & Analysis

- **Impact**: Users may be underpaid or overpaid due to rounding artifacts
- **Exploitability**: Possible if attacker times edge cases or fragments positions
- **Reachability**: Common across staking, bonding, dividend, and token exchange logic
- **Complexity**: Simple math mistake, often in low-attention areas
- **Detectability**: Clear in unit tests or visual audits of amount * X / Y patterns