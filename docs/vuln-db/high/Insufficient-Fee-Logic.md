# Insufficient Fee Logic in DEXs

```YAML
id: TBA
title: Insufficient Fee Logic in DEXs
severity: H
category: economic
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Protocol revenue loss or incentive misalignment due to under-collected fees
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-704
swc: SWC-135
```

## 📝 Description

- Insufficient Fee Logic in DEXs occurs when a decentralized exchange implements flawed or incomplete fee calculation mechanisms, often due to:
- Incorrect math for fee deduction (e.g., using multiplication before division)
- Rounding errors in percentage-based calculations
- Failing to deduct or enforce minimum fee amounts
- Allowing users to bypass fee logic entirely via custom routes or zero-fee paths

## 🚨 Vulnerable Code

```solidity

function swap(uint256 amountIn, address tokenIn, address tokenOut) external {
    uint256 fee = amountIn * 3 / 1000; // ❌ vulnerable to rounding errors
    uint256 amountAfterFee = amountIn - fee;
    // Perform swap using amountAfterFee
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A DEX calculates the fee as amountIn * 3 / 1000, truncating decimals.
2. An attacker sends repeated swaps with amountIn = 1 token.
3. Due to integer rounding, fee = 0 for each trade.
4. By batching small trades, the attacker achieves high-volume routing without paying fees.
5. The protocol loses revenue, and arbitrage bots exploit the system continuously.

**Assumptions:**

- Fee is calculated with integer math and truncates small values.
- There is no minimum fee enforcement (e.g., require(fee > 0)).
- The attacker can interact with the DEX via custom scripts or smart contracts.
- The fee logic is embedded in swap or liquidity provision code and affects economic incentives.

## ✅ Fixed Code

```solidity

function swap(uint256 amountIn, address tokenIn, address tokenOut) external {
    uint256 fee = (amountIn * 3 + 999) / 1000; // ✅ rounding up
    require(fee > 0, "Fee too small");
    uint256 amountAfterFee = amountIn - fee;
    // proceed with amountAfterFee
}
```

## 🛡️ Prevention

### Primary Defenses

- Use fixed-point math libraries (e.g., SafeMath, FixedPoint) to avoid rounding errors.
- Round fees up, not down, and enforce minimum fee thresholds.
- Log and monitor per-trade fee amounts to detect micro-exploit patterns.

### Additional Safeguards

- Apply require(fee > 0) and cap max trades per block or per user.
- Penalize or rate-limit low-fee trades on high-volume addresses.
- Use unit testing and fuzzing to simulate edge cases (1–10 wei trades).

### Detection Methods

- Slither: incorrect-math, insufficient-fee-check
- Manual audit of fee math for truncation or unchecked zero results
- Tools: Echidna, Foundry fuzzing with minimal input sizes

## 🕰️ Historical Exploits

   
- **Name:** SushiSwap Fee Logic Error (Dust Rounding) 
- **Date:** 2020-12 
- **Loss:** Fee calculation for small trades failed to charge users appropriately 
- **Post-mortem:** [Link to post-mortem](https://github.com/sushiswap/sushiswap/issues/71)  

## 📚 Further Reading

- [SWC-135: Incorrect Implementation](https://swcregistry.io/docs/SWC-135) 
- [CWE-704: Incorrect Type Conversion or Cast](https://cwe.mitre.org/data/definitions/704.html) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Insufficient Fee Logic in DEXs
severity: H
score:
impact: 4     
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 3.85
```

---

## 📄 Justifications & Analysis

- **Impact**: Undermines protocol economics, impacting both fees and LP ROI.
- **Exploitability**: Attackers can batch trades or simulate fees to find zero-fee edge cases.
- **Reachability**: Affects core swap paths in most DEX implementations.
- **Complexity**: Moderate; knowledge of math rounding needed.
- **Detectability**: Static analysis, simulation, or monitoring required to detect subtle effects.

