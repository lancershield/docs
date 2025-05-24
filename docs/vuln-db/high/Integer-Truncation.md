# Integer Truncation

```YAML
id: TBA
title: Integer Truncation
severity: H
category: arithmetic
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Value loss or miscalculation due to type downcasting
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-197
swc: SWC-119
```

## 📝 Description

- Integer truncation occurs when a larger integer type (e.g., uint256) is downcasted into a smaller type (e.g., uint8, uint64), causing the loss of high-order bits and resulting in:
- Silent overflows or underflows
- Reward miscalculations
- Token burns or minting misbehavior
- Permission or vote tampering
- In Solidity, downcasting is explicit (e.g., uint8(x)), but often misused in reward loops, cross-contract interfaces, or storage compression.

## 🚨 Vulnerable Code

```solidity

function distributeReward(uint256 amount) external {
    uint8 reward = uint8(amount); // ❌ truncates amount if > 255
    rewardToken.transfer(msg.sender, reward);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A staking system calculates rewards in uint256 and passes them into a reward field stored as uint8.
2. A whale earns 1,200 tokens as a reward.
3. The reward gets truncated: uint8(1200) → 1200 % 256 = 176.
4. The user receives only 176 tokens instead of 1200.
5. The remaining 1024 tokens are effectively lost or stuck.

**Assumptions:**

- Downcasting is used in transfer, loop, or interface logic.
- No bounds check (require(x < 256)) is enforced before casting.
- Amounts may exceed the target type's maximum value.

## ✅ Fixed Code

```solidity

function distributeReward(uint256 amount) external {
    require(amount <= type(uint8).max, "Reward overflow");
    rewardToken.transfer(msg.sender, amount); // ✅ or avoid casting entirely
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid downcasting unless absolutely necessary.
- Use full-width types (uint256) for balances, amounts, and rewards.
- If downcasting, enforce range checks (require(x <= type(uint8).max)).

### Additional Safeguards

- Use fixed-point math libraries like FixedPointMathLib or SafeCast.
- Explicitly define storage layout across contract boundaries.
- Avoid gas micro-optimizations that sacrifice type safety.

### Detection Methods

- Static analysis for uintX(y) casts where X < 256.
- Check for values passed between storage variables of different widths.
- Tools: Slither (incorrect-casting), MythX, Foundry property testing

## 🕰️ Historical Exploits
 
- **Name:** DeFi Options Vault Truncation 
- **Date:** 2022 
- **Loss:** ~$45,000 in under-distributed rewards due to `uint64` to `uint8` cast 
- **Post-mortem:** [Link to post-mortem](https://omniscia.io/) 
  
## 📚 Further Reading

- [SWC-119: Shadowing or Misused Variables](https://swcregistry.io/docs/SWC-119) 
- [CWE-197: Numeric Truncation Error](https://cwe.mitre.org/data/definitions/197.html) 
- [OpenZeppelin SafeCast Docs](https://docs.openzeppelin.com/contracts/4.x/api/utils#SafeCast) 
- [Solidity Docs – Type Conversions](https://docs.soliditylang.org/en/latest/types.html#implicit-conversions) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Integer Truncation
severity: H
score:
impact: 4  
exploitability: 3  
reachability: 4 
complexity: 2   
detectability: 4   
finalScore: 3.65
```

---

## 📄 Justifications & Analysis

- **Impact**: May reduce or destroy rewards and alter fund distributions; accumulated errors cause financial loss.
- **Exploitability**: Feasible with crafted inputs or extreme values.
- **Reachability**: Present across reward, fee, token, and interface boundaries.
- **Complexity**: Simple mistake, often made by developers optimizing storage or using small types.
- **Detectability**: Slither and audits flag unsafe casts effectively.
