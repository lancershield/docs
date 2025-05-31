# Sandwich Attack

```YAML
id: LS32H
title: Sandwich Attack
baseSeverity: H
category: mev
language: solidity
blockchain: [ethereum]
impact: User losses via manipulated slippage and unfair trade execution
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-358
swc: SWC-114
```

## 📝 Description

- A sandwich attack is a form of Miner Extractable Value (MEV) exploitation where an attacker front-runs and back-runs a user's transaction to extract arbitrage profit. 
- This occurs primarily in automated market makers (AMMs) like Uniswap, SushiSwap, or custom DEX contracts when:
- The contract allows trades without slippage protection
- The attacker can observe the pending user transaction in the public mempool
- The protocol exposes price-setting logic before the state update
- In a sandwich attack:
- The attacker places a buy order before the user’s transaction (front-run), pushing the price up.
- The user executes at a worse price.
- The attacker sells immediately after (back-run) to capture the spread.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract SimpleDEX {
    uint256 public tokenReserve = 1000 ether;
    uint256 public ethReserve = 100 ether;

    function buyTokens() external payable {
        require(msg.value > 0, "No ETH sent");

        uint256 tokensOut = (msg.value * tokenReserve) / (ethReserve + msg.value);
        tokenReserve -= tokensOut;
        ethReserve += msg.value;

        // ❌ No slippage control
        _transfer(msg.sender, tokensOut);
    }

    function _transfer(address to, uint256 amount) internal {
        // token transfer logic
    }

    receive() external payable {}
}
```

## 🧪 Exploit Scenario

1. Alice sends a buyTokens() transaction with 10 ETH.
2. The attacker observes this in the mempool.
3. The attacker sends a buyTokens() tx with 100 ETH just before Alice's.
4. Alice’s transaction executes after the price has shifted, so she gets fewer tokens than expected.
5. The attacker back-runs with a sellTokens() to dump at a higher price.
6. Attacker extracts profit at the expense of Alice’s slippage.

**Assumptions:**

- Transaction is publicly visible in mempool.
- No max slippage or deadline set by the user.
- Price-setting logic is exposed to pre-execution analysis.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeDEX {
    uint256 public tokenReserve = 1000 ether;
    uint256 public ethReserve = 100 ether;

    function buyTokens(uint256 minTokensOut) external payable {
        require(msg.value > 0, "No ETH sent");

        uint256 tokensOut = (msg.value * tokenReserve) / (ethReserve + msg.value);
        require(tokensOut >= minTokensOut, "Slippage too high"); // ✅ slippage check

        tokenReserve -= tokensOut;
        ethReserve += msg.value;

        _transfer(msg.sender, tokensOut);
    }

    function _transfer(address to, uint256 amount) internal {
        // token transfer logic
    }

    receive() external payable {}
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Applicable across all AMM-based DeFi transactions where slippage is poorly configured."
- context: "Sophisticated users using Flashbots RPC"
  severity: L
  reasoning: "Attack surface greatly reduced through private mempool routing."
- context: "Mobile/web frontend auto-generated swaps"
  severity: H
  reasoning: "These systems often set loose slippage by default, making users vulnerable."
```

## 🛡️ Prevention

### Primary Defenses

- Require minAmountOut or maxAmountIn parameters in all swap functions.
- Reject trades if slippage exceeds acceptable bounds.

### Additional Safeguards

- Use deadline/expiry timestamps in user-submitted orders.
- Encourage users to route orders through MEV-protected relayers (e.g., Flashbots Protect).
- Consider adding randomized delays or batching to reduce predictability.

### Detection Methods

- Review swap functions for missing slippage bounds or deadlines.
- Simulate MEV attacks in forked testnets.
- Tools: Slither (unbounded-slippage), MythX, Foundry gas differential tests

## 🕰️ Historical Exploits

- **Name:** Uniswap v3 Stablecoin Swap Exploit 
- **Date:** 2025-03-13 
- **Loss:** Trader lost approximately 98% of $220,764 USDC due to a sandwich attack 
- **Post-mortem:** [Link to post-mortem](https://www.ccn.com/education/crypto/sandwich-attack-in-crypto/) 
- **Name:** PEPE Token Sandwich Bot Manipulation 
- **Date:** 2023 
- **Loss:** Investors suffered significant losses due to price manipulation by sandwich bots 
- **Post-mortem:** [Link to post-mortem](https://blockworks.co/news/sandwich-attack-mev-ethereum)
  
## 📚 Further Reading

- [SWC-114: Transaction Ordering Dependence](https://swcregistry.io/docs/SWC-114/) 
- [Flashbots MEV Overview](https://docs.flashbots.net/) 
- [Uniswap Docs – Slippage and Sandwiching](https://docs.uniswap.org/) 

---

## ✅ Vulnerability Report

```markdown
id: LS32H
title: Sandwich Attack
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

- **Impact**: Users overpay or lose funds to MEV bots exploiting slippage.
- **Exploitability**: Mempool bots and priority gas auctions make this easy.
- **Reachability**: Frequent in DEXs and swap logic that lacks slippage bounds.
- **Complexity**: Moderate effort, widely deployed via MEV searchers.
- **Detectability**: Easily audited via static tools and mempool analysis.