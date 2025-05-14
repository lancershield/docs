# Oracle Feed Delay Exploits Enabling Price Manipulation or Front-Running

```YAML
id: TBA
title: Oracle Feed Delay Exploits Enabling Price Manipulation or Front-Running
severity: H
category: oracle-manipulation
language: solidity
blockchain: [ethereum]
impact: Exploiter can use stale prices to drain or manipulate protocol logic
status: draft
complexity: high
attack_vector: oracle
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-200
swc: SWC-119
```

## 📝 Description

- Oracle feed delay exploits occur when a smart contract depends on external oracle prices (e.g., Chainlink, TWAP, custom oracles) that may become outdated or lag behind the current market state. If price updates are delayed, attackers can:
- Trade on stale prices, executing arbitrage against the true market value,
- Borrow against inflated collateral values,
- Trigger liquidations, minting, or redemptions based on incorrect assumptions.

This attack is especially effective when protocols fail to:
- Check oracle data freshness (`timestamp`),
- Use time-weighted or resistant mechanisms,
- Or gate high-impact actions during volatile updates.

## 🚨 Vulnerable Code

```solidity
interface IOracle {
    function getPrice() external view returns (uint256 price, uint256 lastUpdated);
}

contract OracleDelayVulnerable {
    IOracle public priceOracle;

    function buy() external payable {
        (uint256 price, uint256 updatedAt) = priceOracle.getPrice();
        // ❌ No freshness check
        require(msg.value >= price, "Insufficient payment");
        // Mint tokens, etc.
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker observes that getPrice() returns 1 ETH = $2000, but ETH has dropped to $1600.
2. Oracle has not yet updated due to delay or paused feed.
3. Attacker uses 1 ETH to buy tokens at the stale $2000 price.
4. When oracle updates, attacker sells back or exits with net profit.
5. Protocol loses funds or issues assets at overvalued prices.

**Assumptions:**

- Oracle value is outdated or updated infrequently.
- No timestamp or deviation guard in place.
- High-value logic depends directly on the price returned.

## ✅ Fixed Code

```solidity

uint256 public constant MAX_ORACLE_DELAY = 2 minutes;

function buy() external payable {
    (uint256 price, uint256 updatedAt) = priceOracle.getPrice();
    require(block.timestamp - updatedAt <= MAX_ORACLE_DELAY, "Stale price");

    require(msg.value >= price, "Insufficient payment");
    // Proceed with logic
}

```


## 🛡️ Prevention

### Primary Defenses

- Enforce staleness checks using oracle timestamps (e.g., Chainlink updatedAt).
- Use TWAPs, medianizers, or multi-oracle mechanisms to reduce impact of single delays.
- Apply maximum deviation limits: e.g., reject prices that change >30% in a block.

### Additional Safeguards

- Pause protocol actions if oracle becomes unreliable.
- Use Chainlink's AggregatorV3Interface.latestRoundData() with timestamp checks.
- Apply circuit breakers on large transactions based on oracle conditions.

### Detection Methods

- Slither: unchecked-oracle-read, stale-data-dependency, manipulatable-oracle detectors.
- Manual audit of every function that uses oracle pricing without a freshness check.
- Simulation testing with delayed or manipulated oracle feeds.

## 🕰️ Historical Exploits

- **Name:** bZx Oracle Manipulation 
- **Date:** 2020 
- **Loss:** ~$1M 
- **Post-mortem:** [Link](https://blog.bzx.network/postmortem-2-15-20-2f37c4f28a5c) 



## 📚 Further Reading

- [SWC-119: Shadowed State Variables (related to Oracle abuse)](https://swcregistry.io/docs/SWC-119) 
- [Chainlink Docs – Timestamp & Round Check](https://docs.chain.link/data-feeds/api#round-id) 
- [Vitalik – Oracle Problems](https://vitalik.ca/general/2020/07/21/oracles.html) 
- [OpenZeppelin – Oracle Design Patterns](https://docs.openzeppelin.com/contracts/4.x/api/utils#PriceOracle) 


---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Oracle Feed Delay Exploits Enabling Price Manipulation or Front-Running
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

- **Impact**: Major — can drain vaults, misprice mint/burn/redemption, or abuse liquidation.
- **Exploitability**: Depends on timing oracle lag with market volatility.
- **Reachability**: Found in most DeFi protocols with external price dependency.
- **Complexity**: Medium — needs oracle insight or trading bot.
- **Detectability**: High — timestamp checks are well-known best practices.