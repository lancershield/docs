# Stale Price Usage 

```YAML

id: TBA
title: Stale Price Usage Leading to Exploitable Financial Operations
severity: H
category: oracle-manipulation
language: solidity
blockchain: [ethereum]
impact: Attackers exploit outdated prices to manipulate trades, minting, or liquidations
status: draft
complexity: medium
attack_vector: oracle
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-200
swc: SWC-119
```

## 📝 Description

- Stale Price Usage occurs when smart contracts use price data from an oracle or feed without validating its freshness or update timestamp. 
- This allows attackers to exploit outdated rates that no longer reflect the current market, enabling:
- Underpriced minting, 
- Overpriced liquidations, 
- Improper collateralization or redemption, or
- Arbitrage against real-time prices.
- The vulnerability is common in DeFi protocols that rely on Chainlink, TWAPs, or custom oracles but fail to verify the age of the price.

## 🚨 Vulnerable Code

```solidity
interface IOracle {
    function getPrice() external view returns (uint256 price, uint256 lastUpdated);
}

contract StalePriceVulnerable {
    IOracle public priceFeed;

    function buy() external payable {
        (uint256 price, ) = priceFeed.getPrice();
        require(msg.value >= price, "Insufficient payment");
        // Mint token, redeem, etc.
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Oracle’s getPrice() returns ETH = $2000 with lastUpdated from 1 hour ago.
2. Market price drops to $1500.
3. Contract does not check lastUpdated, so stale $2000 price is accepted.
4. Attacker mints tokens or executes trades based on the overvalued ETH rate.
5. Profit is realized at the cost of the protocol or liquidity pool.

**Assumptions:**

- Oracle updates are infrequent or externally triggered.
- Contract uses price without validating lastUpdated.

## ✅ Fixed Code

```solidity

uint256 public constant MAX_DELAY = 60; // 60 seconds

function buy() external payable {
    (uint256 price, uint256 lastUpdated) = priceFeed.getPrice();
    require(block.timestamp - lastUpdated <= MAX_DELAY, "Stale price");

    require(msg.value >= price, "Insufficient payment");
    // Safe to proceed
}
```

## 🛡️ Prevention

### Primary Defenses

- Always validate price freshness using the oracle's updatedAt timestamp.
- Define a maximum allowed delay (e.g., MAX_DELAY = 60 seconds).
- Use trusted oracles like Chainlink, which expose round and timestamp data.

### Additional Safeguards

- Integrate heartbeat monitoring to pause the protocol if oracle feeds stop updating.
- Combine multiple oracles for cross-verification and fallback.
- Monitor for sudden deltas between on-chain price and TWAP/real-time values.

### Detection Methods

- Slither: oracle-no-timestamp-check, stale-price-read, unchecked-oracle-usage detectors.
- Manual audit of any function that calls getPrice(), latestRoundData(), or similar, ensuring timestamp validation is present.
- Simulation testing with delayed oracle updates.

## 🕰️ Historical Exploits

- **Name:** bZx Oracle Exploit 
- **Date:** 2020 
- **Loss:** ~$1M 
- **Post-mortem:** [Link](https://blog.bzx.network/postmortem-2-15-20-2f37c4f28a5c) 

## 📚 Further Reading

- [SWC-119: Unchecked Return Values for Oracle Data](https://swcregistry.io/docs/SWC-119) 
- [Chainlink – Price Feed Security](https://docs.chain.link/data-feeds/security) 
- [Vitalik – Oracle Risks in DeFi](https://vitalik.ca/general/2020/07/21/oracles.html) 


---
## ✅ Vulnerability Report

```markdown
id: TBA
title: Stale Price Usage Leading to Exploitable Financial Operations
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 4  
finalScore: 4.3
```


---

## 📄 Justifications & Analysis

- **Impact**: Enables minting, redeeming, or trading against mispriced assets.
- **Exploitability**: Occurs when attackers observe oracle delay.
- **Reachability**: Present across DeFi protocols with any price dependency.
- **Complexity**: Moderate — attacker needs oracle insight and fast execution.
- **Detectability**: High — easily flagged with timestamp comparison.