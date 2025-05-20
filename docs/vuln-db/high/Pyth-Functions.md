# Deprecated Pyth Functions 

```YAML
id: TBA
title: Deprecated Pyth Functions  
severity: H
category: oracle
language: solidity
blockchain: [ethereum]
impact: Incorrect pricing, failed updates, or exploitable economic assumptions
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-477
swc: SWC-136
```

## 📝 Description

- The Pyth Network provides real-time price feeds to smart contracts via on-chain pull or push mechanisms.
- As Pyth evolves, it deprecates older methods, such as getPriceUnsafe() and getPriceNoOlderThan(), in favor of safer and more robust APIs like parsePriceFeedUpdates(). Continuing to use deprecated methods leads to:
- Insecure reliance on stale or unchecked price data
- Incorrect price assumptions that attackers can exploit
- Upgrade incompatibilities and silent failures during feed changes
- These deprecated functions bypass safety checks, return outdated prices, or fail to incorporate confidence intervals, allowing price manipulation or erroneous liquidations.

## 🚨 Vulnerable Code

```solidity

// ❌ Deprecated interface usage
IPyth pyth = IPyth(0x...);
bytes32 priceId = 0x1234...;

function getPrice() public view returns (int64) {
    PythStructs.Price memory price = pyth.getPriceUnsafe(priceId); // ❌ Deprecated
    return price.price;
}
```

## 🧪 Exploit Scenario

1. A lending platform uses pyth.getPriceUnsafe() to determine collateral value.
2. An attacker front-runs the transaction with a stale or maliciously injected price update.
3. Since no freshness or confidence checks are enforced, the attacker inflates the price.
4. This allows over-borrowing or faulty liquidation at incorrect market prices.

**Assumptions:**

- Contract uses legacy Pyth interfaces for price consumption.
- No manual bounds or confidence checks are added.
- The oracle feed may be slow, unavailable, or manipulated.

## ✅ Fixed Code

```solidity

// ✅ Secure usage via the current recommended interface
function updatePrice(bytes[] calldata priceUpdateData) external payable {
    uint64 minPublishTime = uint64(block.timestamp - 60);
    bytes32 ;
    priceIds[0] = 0x1234...;

    PythStructs.PriceFeed[] memory priceFeeds = pyth.parsePriceFeedUpdates(
        priceUpdateData,
        priceIds,
        minPublishTime,
        block.timestamp
    );

    int64 price = priceFeeds[0].price.price;
    require(priceFeeds[0].price.conf > 0, "Invalid confidence");
    // Use price safely
}
```

## 🛡️ Prevention

### Primary Defenses

- Migrate to current Pyth APIs and deprecate all getPriceUnsafe, getPrice, and getPriceNoOlderThan.
- Require freshness bounds (minPublishTime) and confidence level validation.

### Additional Safeguards

- Limit acceptable deviation from expected prices (oracleDeviationLimit).
- Use fallback oracles in case Pyth feed is delayed or broken.

### Detection Methods

- Search for getPriceUnsafe, getPriceNoOlderThan, and similar legacy calls.
- Verify Pyth version compatibility and monitor update timelines.
- Tools: Slither (custom rule), grep, static analysis on oracle interfaces

## 🕰️ Historical Exploits

- **Name:** Pyth Network Deprecated Function Misuse 
- **Date:** 2023 
- **Loss:** Potential data integrity issues due to outdated function usage 
- **Post-mortem:** [Link to post-mortem](https://docs.pyth.network/home/security)
  
## 📚 Further Reading

- [SWC-136: Unencrypted Sensitive Data](https://swcregistry.io/docs/SWC-136/) 
- [Pyth Network GitHub](https://github.com/pyth-network)
- [Understanding the PYTH Tokenomics – Pyth Network](https://www.pyth.network/blog/understanding-the-pyth-tokenomics) 
- [What is Pyth Network? – Techopedia](https://www.techopedia.com/definition/pyth-network)

---
  
## ✅ Vulnerability Report

```markdown
id: TBA
title: Usage of Deprecated Pyth Functions Leads to Broken Oracle Integrations and Stale Price Feeds
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

- **Impact**: Use of outdated oracle logic can lead to protocol-wide mispricing and loss.
- **Exploitability**: Attackers can manipulate or reuse stale price feeds if safety checks are skipped.
- **Reachability**: High across DeFi platforms using Pyth for price discovery.
- **Complexity**: Easy to misuse due to outdated tutorials or unpatched codebases.
- **Detectability**: Missed unless auditors focus on oracle versioning and SDK updates.