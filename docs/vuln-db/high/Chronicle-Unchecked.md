# Chronicle Unchecked Price

```YAML
id: TBA
title: Chronicle Unchecked Price Usage Enables Stale Feed Exploits and Incorrect Valuations
severity: H
category: oracle
language: solidity
blockchain: [ethereum]
impact: Protocol mispricing, economic manipulation, or fund loss
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-347
swc: SWC-136
```

## 📝 Description

- Chronicle is an on-chain oracle protocol that provides price data via push-based or relay-driven updates. 
- While Chronicle offers functions like getPrice() and getPriceWithTimestamp(), using these functions without validating the freshness or integrity of the returned data introduces critical security risks. These include:
- Consuming stale prices that have not been updated in time
- Trusting values with zero confidence bounds
- Assuming the oracle is always live, without timeout checks
- Unchecked use of Chronicle price data allows attackers to exploit price drift or feed liveness assumptions, particularly in lending, AMMs, or liquidation-sensitive protocols.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IChronicle {
    function getPrice(bytes32 assetId) external view returns (int256);
}

contract VulnerableOracle {
    IChronicle public oracle;
    bytes32 public assetId;

    constructor(IChronicle _oracle, bytes32 _assetId) {
        oracle = _oracle;
        assetId = _assetId;
    }

    function fetchPrice() external view returns (int256) {
        return oracle.getPrice(assetId); // ❌ No freshness check
    }
}
```

## 🧪 Exploit Scenario

1. Chronicle relayers go offline or are delayed for several blocks.
2. A protocol using getPrice() fetches outdated values (e.g., from 10 minutes ago).
3. An attacker manipulates the off-chain price in that time or takes advantage of market drift.
4. They deposit/borrow/withdraw using misaligned prices, draining the protocol via price manipulation or arbitrage.

**Assumptions:**

- Oracle data is not validated for freshness (timestamp or heartbeat missing).
- Price consumers do not cross-check validUntil, conf, or updatedAt values (if available).

## ✅ Fixed Code

```solidity
pragma solidity ^0.8.0;

interface IChronicleSafe {
    function getPriceWithTimestamp(bytes32 assetId) external view returns (int256 price, uint256 timestamp);
}

contract SafeOracle {
    IChronicleSafe public oracle;
    bytes32 public assetId;
    uint256 public maxStaleness = 60; // e.g., 60 seconds

    constructor(IChronicleSafe _oracle, bytes32 _assetId) {
        oracle = _oracle;
        assetId = _assetId;
    }

    function fetchPrice() external view returns (int256) {
        (int256 price, uint256 updatedAt) = oracle.getPriceWithTimestamp(assetId);
        require(block.timestamp - updatedAt <= maxStaleness, "Stale price");
        require(price > 0, "Invalid price");
        return price;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Never trust raw oracle outputs without time-based validation.
- Define a maxStaleness interval and enforce it strictly.

### Additional Safeguards

- Check for zero or negative prices explicitly.
- Use a fallback oracle or heartbeat to validate oracle liveness.

### Detection Methods

- Search for calls to getPrice() or getPrice(...) without timestamp or staleness checks.
- Confirm that consumers validate freshness, confidence, and price bounds.
- Tools: Slither (custom oracle liveness detector), manual review of price integration

## 🕰️ Historical Exploits

- **Name:** Chronicle Price Feed Misuse 
- **Date:** 2024 
- **Loss:** Potential for incorrect pricing due to unchecked oracle data 
- **Post-mortem:** [Link to post-mortem](https://medium.com/%40arthurlabs/smart-contract-vulnerabilities-how-to-audit-your-code-before-launch-1e8190e56be6) 
  
## 📚 Further Reading

- [SWC-136: Unchecked Oracle Values](https://swcregistry.io/docs/SWC-136/) 
- [Chronicle's Price Feed Security Analysis – Messari](https://messari.io/copilot/share/chronicle-s-price-feed-security-analysis-c0ab6a53-4775-4dd6-bd97-1d28fe3e6785) 
- [Slither Detector Documentation – GitHub](https://github.com/crytic/slither/wiki/Detector-Documentation) 
- [Chronicle Oracle Documentation – Chronicle Labs](https://docs.chroniclelabs.org/Resources/FAQ/Oracles#how-do-i-check-if-an-oracle-becomes-inactive-gets-deprecated)
---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Chronicle Unchecked Price Usage Enables Stale Feed Exploits and Incorrect Valuations
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

- **Impact**: Outdated price data can lead to full protocol exploitation through inaccurate valuation.
- **Exploitability**: Attackers can act during oracle downtime or delay periods.
- **Reachability**: Frequent in DeFi lending, AMMs, or NFT appraisals using Chronicle.
- **Complexity**: Simple omission of validation logic.
- **Detectability**: Subtle unless oracle integration is reviewed line-by-line.