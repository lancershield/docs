# Miner Extractable Value 
 
```YAML
id: TBA
title: Miner Extractable Value
severity: H
category: mev
language: solidity
blockchain: [ethereum]
impact: Profitable manipulation of user transactions and protocol logic
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-358
swc: SWC-114
```

## 📝 Description

- Miner Extractable Value (MEV) refers to the ability of a validator, block builder, or any party with mempool visibility to reorder, insert, or censor transactions within a block to extract profit. 
- Smart contracts that depend on user-submitted inputs, lack replay protections, or reveal state before execution are particularly vulnerable to:
- Front-running: attackers copy and preempt the user's transaction
- Back-running: attackers capitalize on state changes after a user’s action
- Sandwich attacks: attackers place one transaction before and one after the victim to extract slippage or arbitrage
- MEV exploits cause loss of user funds, degraded UX, and unfair execution environments for protocols such as DEXes, NFT mints, liquidation engines, and oracle updaters.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract SimpleDEX {
    uint256 public price = 1000;

    function buy() external payable {
        require(msg.value == price, "Incorrect payment");
        // ❌ Attacker can front-run this call to manipulate price externally
        _deliverToken(msg.sender);
    }

    function updatePrice(uint256 newPrice) external {
        price = newPrice; // 🧨 no access control or delay
    }

    function _deliverToken(address user) internal {
        // Token mint or transfer logic
    }
}
```

## 🧪 Exploit Scenario

1. Alice sends a transaction to buy() a token at 1000 wei.
2. Attacker monitors the mempool and sees Alice’s pending tx.
3. The attacker preempts it by sending updatePrice(2000) with higher gas.
4. Alice's transaction is mined second and now fails or overpays.
5. The attacker resets the price afterward to avoid detection.

**Assumptions:**

- Mempool is public (true on Ethereum mainnet)
- No anti-front-running protections are applied
- Transaction ordering is not enforced

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract AntiMEVDEX {
    uint256 public immutable price = 1000;

    mapping(address => uint256) public nonces;

    function buy(uint256 expectedPrice, uint256 nonce) external payable {
        require(expectedPrice == price, "Price changed");
        require(nonces[msg.sender] == nonce, "Invalid nonce");

        nonces[msg.sender] += 1;
        _deliverToken(msg.sender);
    }

    function _deliverToken(address user) internal {
        // Token mint or transfer logic
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use commit-reveal or signature-based meta-transactions to hide intent.
- Require users to commit to state (price, nonce, etc.) in calldata.
- Implement delays or access control on critical state-changing operations.

### Additional Safeguards

- Integrate MEV-aware relayers or private mempool solutions like Flashbots.
- Monitor transactions for reordering patterns and excessive slippage.
- Minimize mutable global state prior to execution.

### Detection Methods

- Audit all publicly callable functions with economic impact.
- Simulate front-running/back-running scenarios in testnet forks.
- Tools: Tenderly trace, Foundry fork-mode tests, Slither (tx.origin, public-mutable-state)

## 🕰️ Historical Exploits

- **Name:** JaredFromSubway.eth Sandwich Bot 
- **Date:** 2023 
- **Loss:** Millions in cumulative trader losses due to over 238,000 sandwich attacks 
- **Post-mortem:** [Link to post-mortem](https://www.coinmetro.com/learning-lab/mev-maximal-extractable-value-explained) 
- **Name:** Curve Finance MEV Block Reward 
- **Date:** 2023-07-31 
- **Loss:** Over $1 million in MEV rewards extracted during DeFi exploit 
- **Post-mortem:** [Link to post-mortem](https://cointelegraph.com/news/ethereum-million-dollar-mev-block-reward-amid-curve-finance-exploit) 
  
## 📚 Further Reading

- [SWC-114: Transaction Ordering Dependence](https://swcregistry.io/docs/SWC-114/)
- [Flashbots – Introduction to MEV](https://docs.flashbots.net/)
- [Solidity Docs – Front-Running Protection](https://docs.soliditylang.org/en/latest/security-considerations.html#front-running) 
- [MEV Explorer](https://explore.flashbots.net/)

--- 

## ✅ Vulnerability Report
```markdown
id: TBA
title: Miner Extractable Value 
severity: H
score:
impact: 4  
exploitability: 3 
reachability: 4  
complexity: 2  
detectability: 4  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Users suffer financial loss, failed transactions, or protocol unfairness.
- **Exploitability**: MEV bots actively monitor the mempool for vulnerable patterns.
- **Reachability**: Any open function with economic impact and no transaction binding is affected.
- **Complexity**: Requires basic MEV bot logic or priority gas auctioning.
- **Detectability**: Easily simulated in forks or flagged with Slither rules.