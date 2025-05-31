#  Unchecked Slippage in Multi-Hop Swaps 

```YAML
id: LS17H
title: Unchecked Slippage in Multi-Hop Swaps 
baseSeverity: H
category: dex-integrations
language: solidity
blockchain: [ethereum]
impact: User receives significantly less tokens than expected in swaps
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.6.0", "<latest"]
cwe: CWE-20
swc: SWC-118
```

## 📝 Description

- Unchecked slippage in multi-hop swaps occurs when decentralized exchange (DEX) integrations perform token swaps across multiple liquidity pools (multi-hop) without enforcing a minimum output threshold (`amountOutMin`). This omission allows:
- Swaps to execute with high slippage during volatile market conditions,
- Sandwich or MEV attacks that extract value from users,
- Receivers to receive far fewer tokens than expected, resulting in silent loss.
- This issue is particularly dangerous when interacting with routers like Uniswap, SushiSwap, or PancakeSwap without setting `amountOutMin` or ignoring the return value.

## 🚨 Vulnerable Code

```solidity
interface IUniswapRouter {
    function swapExactTokensForTokens(
        uint256 amountIn,
        uint256 amountOutMin,
        address[] calldata path,
        address to,
        uint256 deadline
    ) external returns (uint256[] memory amounts);
}

contract SlippageUnsafe {
    IUniswapRouter public router;

    function swap(address[] calldata path, uint256 amountIn) external {
        // ❌ No slippage protection: amountOutMin = 0
        router.swapExactTokensForTokens(amountIn, 0, path, msg.sender, block.timestamp);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. User attempts to swap token A to token C via token B (A→B→C) during low liquidity.
2. Attacker front-runs the swap and manipulates the price of token B.
3. Since amountOutMin = 0, the contract accepts any output and completes the swap.
4. User receives significantly fewer tokens, attacker profits from MEV or arbitrage.

**Assumptions:**

- No minimum output check (amountOutMin) is enforced.
- Path includes volatile or illiquid tokens.
- Slippage or oracle protection is not enforced off-chain.

## ✅ Fixed Code

```solidity

function swap(address[] calldata path, uint256 amountIn, uint256 amountOutMin) external {
    require(amountOutMin > 0, "Invalid slippage threshold");

    router.swapExactTokensForTokens(
        amountIn,
        amountOutMin, // ✅ Enforces expected return
        path,
        msg.sender,
        block.timestamp
    );
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Loss of value through multi-hop slippage can go undetected by users."
- context: "Aggregator contract with dynamic routing"
  severity: C
  reasoning: "Exploiters can insert illiquid tokens into path to drain value stealthily."
- context: "Trusted frontend with enforced slippage checks"
  severity: M
  reasoning: "User loss minimized when frontend or SDK sets tight bounds."
```

## 🛡️ Prevention

### Primary Defenses

- Always require user-supplied or computed amountOutMin based on off-chain price feeds or simulations.
- Validate slippage tolerances (e.g., 1%, 0.5%) based on estimated route output.
- Consider quoting with getAmountsOut() before executing the swap.

### Additional Safeguards

- Add UI warnings and slippage selectors in dApp frontend.
- Integrate oracle feeds to detect price deviation from real-time market data.
- Use TWAP oracles or guarded pricing mechanisms in critical contracts.

### Detection Methods

- Slither: dex-integration-no-slippage-check, zero-min-amount, swap-unchecked detectors.
- Manual audit of all swapExactTokensForTokens, swapExactETHForTokens, or similar functions.
- Fuzz testing with manipulated DEX state to simulate low-liquidity attacks.

## 🕰️ Historical Exploits

- **Name:** Level Finance Router Slippage Bug 
- **Date:** 2023 
- **Impact:** $1.1M loss via multi-hop swap route with zero slippage protection 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/level-finance-rekt/) 

## 📚 Further Reading

- [SWC-118: Incorrect Data Validation](https://swcregistry.io/docs/SWC-118) 
- [Uniswap V2 Docs – Slippage Protection](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#swapexacttokensfortokens) 
- [Slither Slippage Check Rules](https://github.com/crytic/slither)

---

## ✅ Vulnerability Report 

```markdown
id: LS17H
title: Unchecked Slippage in Multi-Hop Swaps 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 3   
complexity: 2     
detectability: 4  
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: High — causes immediate value loss due to bad pricing.
- **Exploitability**: High — easily targeted by bots when no slippage guard is present.
- **Reachability**: Seen in custom routers, farm rewards, and auto-compounders.
- **Complexity**: Low — typical oversight during integration.
- **Detectability**: High — audit tools flag this pattern, and it is visually evident.