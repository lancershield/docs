# Chainlink Feed Registry

```YAML
id: TBA
title: Chainlink Feed Registry 
severity: H
category: oracle
language: solidity
blockchain: [ethereum]
impact: Mispriced assets, incorrect logic, or economic manipulation
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.8.0", "<=0.8.25"]
cwe: CWE-352
swc: SWC-136
```

## 📝 Description

- The Chainlink Feed Registry provides a unified interface for fetching multiple Chainlink price feeds using asset pair identifiers. 
- However, developers misusing the registry without validating feed freshness or decimals may introduce critical oracle-related vulnerabilities, such as:
- Consuming stale prices without checking the updatedAt timestamp
- Failing to normalize prices due to ignored decimals()
- Using the wrong base/quote pair (e.g., USD/ETH instead of ETH/USD)
- Ignoring phaseId changes or feed migration logic
- This may result in incorrect asset valuation, over/under-collateralization, faulty rewards, or arbitrage vulnerabilities.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IFeedRegistry {
    function latestRoundData(address base, address quote)
        external
        view
        returns (
            uint80,
            int256 answer,
            uint256,
            uint256 updatedAt,
            uint80
        );
}

contract OracleConsumer {
    IFeedRegistry public registry;
    address public constant ETH = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;
    address public constant USD = 0x0000000000000000000000000000000000000348;

    constructor(address _registry) {
        registry = IFeedRegistry(_registry);
    }

    function getETHPrice() external view returns (int256) {
        (, int256 price, , , ) = registry.latestRoundData(ETH, USD); // ❌ No freshness or decimals check
        return price;
    }
}
```

## 🧪 Exploit Scenario

1. A protocol uses getETHPrice() to determine the value of user deposits.
2. The Chainlink feed for ETH/USD halts updates or stalls due to oracle downtime.
3. The contract continues accepting deposits based on outdated or incorrect prices.
4. An attacker takes advantage of this to deposit overvalued assets and withdraw undervalued ones or avoid liquidation.

**Assumptions:**

- Protocol does not use updatedAt to verify price staleness.
- Price is used directly without applying decimals() scaling or sanity bounds.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

interface IFeedRegistrySafe is IFeedRegistry {
    function decimals(address base, address quote) external view returns (uint8);
}

contract SafeOracleConsumer {
    IFeedRegistrySafe public registry;
    address public constant ETH = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;
    address public constant USD = 0x0000000000000000000000000000000000000348;
    uint256 public maxStaleness = 60; // e.g., 60 seconds

    constructor(address _registry) {
        registry = IFeedRegistrySafe(_registry);
    }

    function getETHPrice() external view returns (uint256) {
        (, int256 price, , uint256 updatedAt, ) = registry.latestRoundData(ETH, USD);
        require(block.timestamp - updatedAt <= maxStaleness, "Stale price");
        require(price > 0, "Invalid price");

        uint8 decimals = registry.decimals(ETH, USD);
        return uint256(price) * 1e18 / (10 ** decimals); // Normalize to 18 decimals
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always check updatedAt and enforce a maximum age
- Normalize prices using decimals(base, quote)
- Sanity-check returned values (e.g., price > 0, within range)

### Additional Safeguards

- Use fallback oracles in case Chainlink stalls
- Enforce admin control for updating feed references dynamically

### Detection Methods

- Search for use of latestRoundData() without updatedAt checks
- Look for direct use of int256 price without decimals() normalization
- Tools: Slither (oracle-check), MythX, manual oracle integration review

## 🕰️ Historical Exploits

- **Name:** Moonwell Chainlink Feed Registry Misconfiguration
- **Date:** 2023-07 
- **Loss:** Potential for incorrect pricing due to misconfigured price feeds 
- **Post-mortem:** [Link to post-mortem](https://github.com/code-423n4/2023-07-moonwell-findings/issues/189)
  
## 📚 Further Reading

- [SWC-136: Unchecked Oracle Values](https://swcregistry.io/docs/SWC-136/) 
- [Chainlink Feed Registry Docs](https://docs.chain.link/data-feeds/feed-registry) 
- [OpenZeppelin Oracle Best Practices](https://docs.openzeppelin.com/contracts/4.x/api/utils#SafeCast) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Chainlink Feed Registry 
severity: H
score:
impact: 4        
exploitability: 3
reachability: 4   
complexity: 2 
detectability: 3 
finalScore: 3.45
```

---

## 📄 Justifications & Analysis

- **Impact**: Misused prices can cause serious financial imbalances or protocol insolvency.
- **Exploitability**: Relies on downtime or feed staleness—realistic in congested oracles.
- **Reachability**: Widely used via FeedRegistry, especially in multi-asset protocols.
- **Complexity**: The vulnerability arises from forgetting to check timestamps or scale values.
- **Detectability**: Easily missed unless explicitly testing oracle staleness and normalization.
