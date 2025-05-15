# Price Oracle Manipulation

```YAML
id: TBA
title: Price Oracle Manipulation
severity: C
category: oracle-manipulation
language: solidity
blockchain: [ethereum]
impact: Mispriced trades, loans, and liquidations
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-345
swc: SWC-114
```

## 📝 Description

- Price Oracle Manipulation occurs when an attacker influences the data source a smart contract relies on for pricing assets.
- If the oracle uses manipulable inputs—such as on-chain DEX prices, thin liquidity pools, or manipulable off-chain data feeds—an attacker can trick the contract into believing an incorrect asset price.
- This results in faulty calculations in lending protocols, AMMs, liquidations, and more.

## 🚨 Vulnerable Code

```solidity
interface IPriceFeed {
    function getPrice(address token) external view returns (uint);
}

contract Lending {
    IPriceFeed public oracle;
    mapping(address => uint256) public collateral;

    constructor(address _oracle) {
        oracle = IPriceFeed(_oracle);
    }

    function borrow(address token, uint256 amount) external {
        uint256 price = oracle.getPrice(token);
        require(collateral[msg.sender] * price >= amount, "Undercollateralized");
        // Issue loan logic
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker identifies that the price feed uses a low-liquidity AMM pair.
2. Attacker performs large buy/sell to skew the price of the token.
3. Calls borrow() using artificially inflated collateral value.
4. Withdraws loaned tokens at the manipulated price, then reverses the price manipulation.

**Assumptions:**

- Oracle is based on TWAP or spot price of an AMM like Uniswap with low liquidity.

- Borrow/loan logic relies directly on unverified price inputs.

## ✅ Fixed Code

```solidity

contract SaferLending {
    IPriceFeed public chainlinkOracle;

    constructor(address _oracle) {
        chainlinkOracle = IPriceFeed(_oracle);
    }

    function borrow(address token, uint256 amount) external {
        uint256 price = chainlinkOracle.getPrice(token); // Uses secure, external oracle
        require(price > 0, "Invalid price");
        // Collateral logic...
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use decentralized oracles like Chainlink, Redstone, or UMA.
- Rely on TWAPs with sufficient duration to avoid short-term manipulation.
- Avoid direct use of low-liquidity AMM pairs as price feeds.

### Additional Safeguards

- Add circuit breakers or sanity checks on extreme price changes.
- Use multi-oracle aggregation with median or weighted average logic.
- Consider governance delays for price-sensitive operations.

### Detection Methods

- Analyze oracle dependency and source trustworthiness.
- Use Slither detectors for untrusted-oracle and price-manipulation.
- Simulate oracle price impact with fuzzing or adversarial testing.

## 🕰️ Historical Exploits

- **Name:** Synthetix sKRW Oracle Exploit
- **Date:** 2019-06-25
- **Loss:** $1B (reversed)
- **Post-mortem:** [Link to post-mortem](https://medium.com/synthetix-blogsynthetix-exchange-oracle-incident-post-mortem-319d54f47c7c)
- **Name:** Harvest Finance Exploit
- **Date:** 2020-10-26
- **Loss:** ~$33M
- **Post-mortem:** [Link to post-mortem](https://medium.com/harvest-finance/harvest-incident-report-1c8e5c590920)

- **Name:** Mango Markets Manipulation
- **Date:** 2022-10-11
- **Loss:** ~$114M
- **Post-mortem:** [Link to post-mortem](https://twitter.com/MangoMarkets/status/1580041892085970945)

## 📚 Further Reading

- [SWC-114: Oracle Manipulation](https://swcregistry.io/docs/SWC-114)
- [Chainlink – Preventing Oracle Attacks](https://blog.chain.link/defi-attacks-smart-contract-vulnerabilities/)
- [Paradigm Research – Oracle Manipulation in DeFi](https://research.paradigm.xyz/OracleManipulation)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Price Oracle Manipulation
severity: C
score:
impact: 5  
exploitability: 5
reachability: 4  
complexity: 3  
detectability: 3  
finalScore: 4.5
```

---

## 📄 Justifications & Analysis

- **Impact**: Can directly lead to protocol insolvency or massive bad debt.

- **Exploitability**: Commonly exploitable on thin-liquidity pairs or unverified TWAPs.

- **Reachability**: Oracle data is usually directly queried in user-facing functions.

- **Complexity**: Requires moderate setup (manipulate AMM pool or data feed).

- **Detectability**: Detectable in code reviews, but often missed in data flow audits.
