# Stuck Swap Pools

```YAML
id: LS44M
title: Stuck Swap Pools
baseSeverity: M
category: liquidity
language: solidity
blockchain: [ethereum]
impact: Permanent or prolonged trading disablement
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: medium
versions: [">0.6.0", "<0.8.25"]
cwe: CWE-664
swc: SWC-131
```
## 📝 Description

- Stuck swap pools occur when Automated Market Makers (AMMs) or liquidity contracts enter a state where further swaps, deposits, or withdrawals are impossible, typically due to:
- Imbalanced reserves, Precision loss or math overflows in the pricing formula,Token transfer failures ,Paused or non-upgradable pool logic preventing recovery.
- This leads to a frozen liquidity pool, user frustration, and potential protocol revenue loss.

## 🚨 Vulnerable Code

```solidity
function swap(uint256 amountIn, address tokenIn, address tokenOut) external {
    uint256 amountOut = getAmountOut(amountIn, tokenIn, tokenOut);
    require(amountOut > 0, "Insufficient output");

    IERC20(tokenIn).transferFrom(msg.sender, address(this), amountIn);
    IERC20(tokenOut).transfer(msg.sender, amountOut); // ❌ Can fail silently (non-returning)
}
```

## 🧪 Exploit Scenario

Step-by-step:

1. A user swaps large amounts of a stablecoin for a volatile token.
2. The volatile token fails to transfer (e.g., due to a deflationary fee or ERC20 non-compliance).
3. The pool records the swap, reducing internal reserves, but no tokens are delivered.
4. The pool has invalid reserves,Future swaps are mispriced or fail,liquidity providers cannot remove their tokens safely.

**Assumptions:**

- The swap pool smart contract does not include fallback mechanisms for failed withdrawals.
- If the smart contract is vulnerable to reentrancy attacks, it may lock tokens during simultaneous interactions.

## ✅ Fixed Code

```solidity

function swap(uint256 amountIn, address tokenIn, address tokenOut) external {
    require(amountIn > 0, "Zero input");

    uint256 balanceBefore = IERC20(tokenOut).balanceOf(address(this));

    IERC20(tokenIn).transferFrom(msg.sender, address(this), amountIn);
    uint256 amountOut = getAmountOut(amountIn, tokenIn, tokenOut);
    require(amountOut > 0, "Insufficient output");

    bool success = IERC20(tokenOut).transfer(msg.sender, amountOut);
    require(success, "Token transfer failed");

    // ✅ Optional: Check final balance
    uint256 balanceAfter = IERC20(tokenOut).balanceOf(address(this));
    require(balanceBefore - balanceAfter == amountOut, "Token accounting mismatch");
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Common risk in AMMs but often recoverable with admin/manual rebalance."
- context: "High-volume public DEX"
  severity: H
  reasoning: "Traders may get stuck, harming UX and possibly enabling manipulation."
- context: "Private swap pool with few users"
  severity: L
  reasoning: "Limited harm, but could still disrupt liquidity provision."
```

## 🛡️ Prevention

### Primary Defenses

- Always check return values from token transfer() and transferFrom().
- Use SafeERC20 from OpenZeppelin for compatibility with non-standard ERC20s.
- Add reserve sanity checks before and after swaps.
- Validate against fee-on-transfer, rebase, and ERC777 hooks where applicable.

### Additional Safeguards

- Implement emergency withdrawal mechanisms for LPs.
- Use a circuit breaker if reserve imbalance crosses a safe threshold.
- Add monitoring for stuck swaps: repeated reverts or output = 0.

### Detection Methods

- Slither: unchecked-transfer, reserve-mismatch, swap-deadlock detectors.
- Manual simulation of edge cases with:
- Tokens returning false,Fee-on-transfer tokens,Token with internal hooks (ERC777, rebase).
- Fuzz testing with various token behaviors.

## 🕰️ Historical Exploits

- **Name:** SushiSwap Liquidity Pool Bug 
- **Date:** April 9, 2023 
- **Loss:** $3.3 million 
- **Post-mortem:** [Link to post-mortem](https://www.certik.com/resources/blog/post-mortem-sushiswap)

## 📚 Further Reading

- [SWC-136: Unexpected Behavior Due to Unverified Assumptions](https://swcregistry.io/docs/SWC-136) 
- [OpenZeppelin SafeERC20 Guide](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20) 
- [Balancer Technical Docs on Pool Accounting](https://docs.balancer.fi/)
- [Uniswap V2 Math Audit Notes](https://uniswap.org) 

----

## ✅ Vulnerability Report

```markdown
id: LS44M
title: Stuck Swap Pools 
severity: M
score:
impact: 5         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Critical — frozen liquidity breaks economic guarantees.
- **Exploitability**: Medium — not always malicious, but common via broken tokens or math.
- **Reachability**: Common in forks or AMM clones with minor swaps.
- **Complexity**: Simple to fix, devastating if ignored.
- **Detectability**: High — Slither and test coverage highlight this quickly.