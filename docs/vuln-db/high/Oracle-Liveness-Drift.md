# Oracle Liveness Drift 

```YAML
id: LS36H
title: Oracle Liveness Drift 
baseSeverity: H
category: oracle
language: solidity
blockchain: [ethereum, arbitrum, optimism, avalanche, bsc]
impact: Price manipulation, arbitrage, or incorrect liquidation
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-200
swc: SWC-114
```

## 📝 Description

- Oracle liveness drift occurs when price data remains unchanged past its expected update interval, leading to stale, inaccurate, or manipulable price feeds. This is common in:
- Cross-chain oracle setups (e.g., Chainlink CCIP, LayerZero relays)
- Off-chain price push mechanisms (e.g., keepers, Pyth pull models)
- Custom oracles that lack on-chain freshness enforcement
- If contracts do not validate oracle timestamps or update frequency, they may:
- Execute trades at outdated prices
- Allow under-collateralized borrowing
- Trigger or skip liquidations
- Be exploited via multi-block arbitrage (e.g., flashloans)

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface Oracle {
    function latestAnswer() external view returns (int256);
}

contract LivenessDriftVault {
    Oracle public priceOracle;

    function getPrice() public view returns (int256) {
        return priceOracle.latestAnswer(); // ❌ no check on staleness or timestamp
    }

    function liquidate(address user) external {
        int256 price = getPrice();
        // liquidate user if price is below threshold (logic omitted)
    }
}
```

## 🧪 Exploit Scenario

1. The oracle stops updating due to relayer downtime or RPC issues.
2. The attacker borrows at full collateralization and dumps the collateral asset, knowing liquidation logic won't adjust.
3. Alternatively, the attacker triggers forced liquidation using a stale high price, even though the real market value dropped.
4. Cross-chain deployments (e.g., BSC ↔ Ethereum) using async oracles have different price states across chains.

**Assumptions:**

- Oracle has drifted or paused without alert.
- Consumer contract doesn't validate freshness.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

interface TimestampedOracle {
    function latestAnswer() external view returns (int256);
    function latestTimestamp() external view returns (uint256);
}

contract SafeOracleVault {
    TimestampedOracle public oracle;
    uint256 public staleThreshold = 300; // 5 minutes

    function getSafePrice() public view returns (int256) {
        require(block.timestamp - oracle.latestTimestamp() <= staleThreshold, "Stale oracle");
        return oracle.latestAnswer();
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Stablecoin protocol or lending platform"
  severity: H
  reasoning: "Delayed updates lead to real financial losses via mispriced actions"

- context: "Read-only dashboard or historical viewer"
  severity: L
  reasoning: "No financial consequences from stale display"

- context: "Oracle with bounded round liveness checks + freshness threshold"
  severity: M
  reasoning: "Partially mitigated, but still exploitable during drift windows"
```

## 🛡️ Prevention

### Primary Defenses

- Enforce freshness checks on every oracle access
- Validate both answer and timestamp/roundId

### Additional Safeguards

- Use fallback oracles (multi-feed model)
- Emit drift-related metrics to external monitoring systems
- Automatically pause protocol actions (e.g., liquidations, trading) on stale oracles

### Detection Methods

- Review for oracle consumers that don’t use latestTimestamp(), latestRoundData(), or equivalent
- Tools: Manual audit, Slither custom rule, static analyzers

## 🕰️ Historical Exploits

- **Name:** KiloEX Oracle Manipulation Exploit 
- **Date:** 2025-04-16 
- **Loss:** ~$7.5 million  
- **Post-mortem:** [Link to post-mortem](https://dig.watch/updates/kiloex-loses-7-5-million-in-oracle-hack) 
- **Name:** Drift Protocol Oracle Desynchronization Bug 
- **Date:** 2022-05-11 
- **Loss:** ~$11.75 million  
- **Post-mortem:** [Link to post-mortem](https://driftprotocol.medium.com/drift-protocol-technical-incident-report-2022-05-11-eedea078b6d4)
-  **Name:** Radiant Capital Cross-Chain Oracle Drift Attack
-  **Date:** 2024-10 
-  **Loss:** ~$58 million 
-  **Post-mortem:** [Link to post-mortem](https://getfailsafe.com/failsafe-web3-security-report-2025/)
  
## 📚 Further Reading

- [Chainlink Docs – LatestRoundData](https://docs.chain.link/data-feeds/api-reference#latestrounddata)
- [SWC-114: Transaction Ordering Dependence](https://swcregistry.io/docs/SWC-114/)
- [OpenZeppelin Defender – Oracle Monitoring](https://docs.openzeppelin.com/defender/) 

---
  
## ✅ Vulnerability Report

```markdown
id: LS36H
title: Oracle Liveness Drift 
severity: H
score:
impact: 5  
exploitability: 3 
reachability: 5  
complexity: 2     
detectability: 4  
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Critical price-based actions execute on outdated data, risking large losses
- **Exploitability**: Occurs during oracle downtime or relayer stalls
- **Reachability**: Widespread across lending, AMMs, derivatives
- **Complexity**: Simple — just wait for oracle to stall
- **Detectability**: Easily flagged by monitoring or testing for stale timestamps