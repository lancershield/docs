# Imbalanced Liquidity

```YAML
id: LS38H
title: Imbalanced Liquidity 
baseSeverity: H
category: amm-liquidity
language: solidity
blockchain: [ethereum, arbitrum, optimism, avalanche, polygon, bsc]
impact: Value leakage, frontrun withdrawals, or pool destabilization
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-670
swc: SWC-131
```

## 📝 Description

- In liquidity pools that support multi-asset deposits or withdrawals (e.g., Curve, Balancer, stable AMMs), imbalanced liquidity occurs when:
- Users deposit or withdraw assets in an asymmetric manner (e.g., only DAI in a DAI/USDC/USDT pool)
- The protocol does not adjust pricing curves or pool shares to penalize imbalanced actions
- If no protections exist (e.g., slippage loss, fee curves, invariant checks), attackers can:
- Withdraw more value than deposited by abusing the pool imbalance
- Arbitrage between tokens before or after interacting with the pool
- Exploit single-sided liquidity additions for economic advantage

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract ImbalancedPool {
    mapping(address => uint256) public balancesDAI;
    mapping(address => uint256) public balancesUSDC;
    uint256 public totalDAI;
    uint256 public totalUSDC;

    function depositDAI(uint256 amount) external {
        balancesDAI[msg.sender] += amount;
        totalDAI += amount;
        // ❌ no consideration for USDC imbalance
    }

    function withdrawProportional() external {
        uint256 share = balancesDAI[msg.sender] / totalDAI;
        uint256 usdcOut = totalUSDC * share; // ❌ attacker gets outsized USDC if DAI dominates
        totalUSDC -= usdcOut;
        // send USDC to msg.sender (omitted)
    }
}
```

## 🧪 Exploit Scenario

1. An attacker deposits 1,000,000 DAI into a pool with 1 DAI and 1,000,000 USDC.
2. They now own ~99.99% of the pool.
3. They call withdrawProportional() and receive ~999,990 USDC.
4. The pool is drained of USDC, and DAI remains over-concentrated.
5. Arbitrageurs capitalize on the imbalance, further degrading the pool.

**Assumptions:**

- No invariant (e.g., constant product or stable curve) protects multi-asset ratio
- No slippage penalty or withdrawal fee is applied to unbalanced exits

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract BalancedPool {
    mapping(address => uint256) public shares;
    uint256 public totalShares;
    uint256 public totalDAI;
    uint256 public totalUSDC;

    function deposit(uint256 daiIn, uint256 usdcIn) external {
        require(daiIn > 0 && usdcIn > 0, "Must deposit both assets");
        uint256 newShares = daiIn + usdcIn; // Simplified invariant
        shares[msg.sender] += newShares;
        totalShares += newShares;
        totalDAI += daiIn;
        totalUSDC += usdcIn;
    }

    function withdraw(uint256 shareAmount) external {
        require(shareAmount <= shares[msg.sender], "Too much");

        uint256 daiOut = (totalDAI * shareAmount) / totalShares;
        uint256 usdcOut = (totalUSDC * shareAmount) / totalShares;

        shares[msg.sender] -= shareAmount;
        totalShares -= shareAmount;
        totalDAI -= daiOut;
        totalUSDC -= usdcOut;

        // send daiOut and usdcOut to user (omitted)
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "DEX or oracle system relies on pool balance for price feeds"
  severity: H
  reasoning: "Attackers can manipulate price and extract profit or oracle control"
- context: "Liquidity pool not connected to external price systems"
  severity: M
  reasoning: "Users suffer slippage, but no oracle exposure"
- context: "Only cosmetic imbalance or managed via rebalance bots"
  severity: L
  reasoning: "No direct impact on functionality"
```

## 🛡️ Prevention

### Primary Defenses

- Apply invariant-based math (e.g., constant sum/product, stable-swap equations)
- Reject or penalize single-sided deposits or withdrawals

### Additional Safeguards

- Add withdrawal fees for imbalanced exits
- Track per-token accounting per user (e.g., Curve’s LP share weighting)
- Use slippage tolerances and simulate impact before confirming

### Detection Methods

- Identify liquidity functions with unconstrained ratio logic
- Test protocol under extreme imbalance conditions
- Tools: Manual review, invariant testing (Foundry, Echidna), Slither

## 🕰️ Historical Exploits

- **Name:** Uranium Finance Exploit 
- **Date:** 2021-04-28 
- **Loss:** ~$50 million stolen due to incorrect token ratio in liquidity pool initialization 
- **Post-mortem:** [Link to post-mortem](https://medium.com/immunefi/building-a-poc-for-the-uranium-heist-ec83fbd83e9f) 
 
## 📚 Further Reading

- [SWC-131: Incorrect Calculation](https://swcregistry.io/docs/SWC-131/)  
- [Balancer Technical Docs – Multi-Token Pools](https://docs.balancer.fi/) 

--- 

## ✅ Vulnerability Report

```markdown
id: LS38H
title: Imbalanced Liquidity 
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

- **Impact**: Pools may be drained or users receive outsized value
- **Exploitability**: Easily triggered by manipulating deposit/withdraw pattern
- **Reachability**: Present in many AMMs or vaults without invariant enforcement
- **Complexity**: Conceptually simple to abuse once mechanics are known
- **Detectability**: Highly visible in code and sim testing, but may be overlooked