# Gas Fee Arbitrage

```YAML
id: TBA
title: Gas Fee Arbitrage 
baseSeverity: M
category: economic
language: solidity
blockchain: [ethereum]
impact: MEV exploitation, gas griefing, or unexpected fund loss
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-406
swc: SWC-131
```

## 📝 Description

- Gas fee arbitrage refers to exploiting differences in gas cost, gas refunds, and transaction ordering to extract economic value from smart contracts. Attackers can:
- Craft transactions that manipulate state just to trigger costly logic (e.g., expensive SSTORE or SELFDESTRUCT)
- Front-run or sandwich legitimate users with low-cost, high-return gas operations
- Exploit gas refund loopholes to get partial gas back while still affecting state
- Inflate gas usage to cause denial-of-service or discourage interaction
- Protocols that tie logic execution or economic outcomes to gas usage (e.g., reward calculations, token issuance per gas burnt) are especially vulnerable.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract GasReward {
    mapping(address => uint256) public lastBlockUsed;

    function claimReward() external {
        require(block.number > lastBlockUsed[msg.sender], "Already claimed");

        uint256 gasStart = gasleft();
        // expensive computation (could be griefed)
        for (uint i = 0; i < 1000; i++) {
            assembly { mstore(0x0, i) }
        }

        uint256 gasUsed = gasStart - gasleft();
        uint256 reward = gasUsed * tx.gasprice; // ❌ ties reward to gas cost
        payable(msg.sender).transfer(reward);
        lastBlockUsed[msg.sender] = block.number;
    }

    receive() external payable {}
}
```

## 🧪 Exploit Scenario

1. A user identifies a contract that refunds gas or pays users based on gasUsed * gasprice.
2. They craft a transaction with:
3. Large loops or dummy state writes to increase gasUsed
4. High gasprice with base fee control under EIP-1559
5. They trigger claimReward() repeatedly, draining ETH from the contract.
6. In MEV scenarios, this can be sandwiched around user transactions or bundled into private blocks for profit.

**Assumptions:**

- Contract rewards or penalizes based on gasUsed or tx.gasprice
- No upper bounds, caps, or sanity checks are enforced

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeGasReward {
    mapping(address => uint256) public lastBlockUsed;

    uint256 public constant MAX_REWARD = 0.01 ether;

    function claimReward() external {
        require(block.number > lastBlockUsed[msg.sender], "Already claimed");
        uint256 reward = MAX_REWARD; // ✅ fixed cap
        payable(msg.sender).transfer(reward);
        lastBlockUsed[msg.sender] = block.number;
    }

    receive() external payable {}
}
```

## 🧭 Contextual Severity

```yaml
- context: "Protocol uses gas reimbursement with no refund or abuse protection"
  severity: M
  reasoning: "Abuse results in token drain or unsustainable incentive model"
- context: "Gas reimbursements capped, validated, or audited"
  severity: L
  reasoning: "Exposure significantly reduced by proper accounting"
- context: "No gas-based rewards or all txs use `msg.sender` payment"
  severity: I
  reasoning: "No vector for abuse exists"
```

## 🛡️ Prevention

### Primary Defenses

- Avoid rewarding based on gasUsed, tx.gasprice, or similar low-level metrics.
- Cap dynamic computations with MAX_REWARD, MAX_GAS_USED, etc.

### Additional Safeguards

- Use Chainlink feeds or off-chain logic for reward rates.
- Add gas usage audits or runtime enforcement of limits.
- Implement EIP-3529 awareness (gas refund limitations)

### Detection Methods

- Look for tx.gasprice, gasleft(), and gas-based calculations in financial logic.
- Simulate transactions with artificially inflated gasprice or block congestion.
- Tools: Slither (gas-related logic), MythX, fuzzing

## 🕰️ Historical Exploits

- **Name:** GasToken Refund Exploit 
- **Date:** 2019–2021 
- **Loss:** Millions in arbitrage via `SELFDESTRUCT` refund farming 
- **Post-mortem:** [Link to post-mortem](https://gastoken.io)

## 📚 Further Reading

- [SWC-131: Gas Griefing](https://swcregistry.io/docs/SWC-131/)
- [EIP-2028: Calldata Gas Cost Reduction](https://eips.ethereum.org/EIPS/eip-2028)  
- [Trail of Bits: Gas-Based Vulnerabilities](https://blog.trailofbits.com/)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Gas Fee Arbitrage 
severity: H
score:
impact: 4  
exploitability: 3 
reachability: 3 
complexity: 2   
detectability: 4  
finalScore: 3.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Allows attacker to extract value or deny access by manipulating gas.
- **Exploitability**: Feasible via smart transaction design or private bundles.
- **Reachability**: Found in game loops, gas refunds, reward multipliers, etc.
- **Complexity**: Moderate—no protocol break needed, just smart gas manipulation.
- **Detectability**: Easy to catch with static tools or code audit.