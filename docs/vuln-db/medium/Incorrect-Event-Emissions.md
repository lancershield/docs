# Incorrect Event Emissions

```YAML
id: TBA
title: Incorrect Event Emissions
severity: M
category: observability
language: solidity
blockchain: [ethereum]
impact: Inconsistent off-chain state / misreported user actions
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-627
swc: SWC-136
```

## 📝 Description

- Solidity smart contracts emit events to allow off-chain systems like subgraphs, explorers, indexers, or governance dashboards to track state changes. If events are emitted with:
- Incorrect parameters (e.g., from, to, amount)
- At the wrong place in the function (before revert, outside conditional)
- Missing entirely from critical flows
- it leads to false historical records, broken analytics, or even governance exploits where decisions rely on emitted logs.
- While this doesn’t affect on-chain state directly, it can severely disrupt systems relying on logs, including token trackers, DEX aggregators, staking dashboards, and bridges.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Token {
    mapping(address => uint256) public balances;

    event Transfer(address indexed from, address indexed to, uint256 value);

    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient");
        balances[msg.sender] -= amount;
        balances[to] += amount;

        emit Transfer(msg.sender, to, amount + 1); // ❌ incorrect value emitted
    }
}
```

## 🧪 Exploit Scenario

1. A DeFi dashboard queries Transfer logs to update token balances.
2. Due to incorrect emission, it records a transfer of 101 tokens instead of 100.
3. A staking aggregator allows users to claim based on logs, resulting in over-distribution of rewards.
4. Governance snapshot or DAO voting weights based on event logs become inaccurate.

**Assumptions:**

- Off-chain systems rely heavily on emitted events for accounting or control
- Events are malformed or misplaced

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeToken {
    mapping(address => uint256) public balances;

    event Transfer(address indexed from, address indexed to, uint256 value);

    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient");
        balances[msg.sender] -= amount;
        balances[to] += amount;

        emit Transfer(msg.sender, to, amount); // ✅ correct value
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Assumes frontend and analytics rely on events, leading to user-facing bugs."
- context: "DeFi Protocol with subgraph dependency"
  severity: H
  reasoning: "Inaccurate event emissions can break critical tracking like staking, balances, or votes."
- context: "Isolated backend contract with no external log usage"
  severity: L
  reasoning: "Issue may go unnoticed if logs are unused or internal-only."
```

## 🛡️ Prevention

### Primary Defenses

- Use automated tools or test assertions to compare emitted logs to state
- Follow “emit-after-state” pattern and avoid emissions inside complex conditionals or try/catch

### Additional Safeguards

- Include emit testing in unit tests
- Avoid emitting logs that imply successful operations before reverting logic

### Detection Methods

- Look for differences between emitted values and state mutations
- Tools: Slither (incorrect-event-arguments), Foundry log assertions, MythX

## 🕰️ Historical Exploits

- **Name:** Lendf.Me Event-Driven Liquidation Drift 
- **Date:** 2020-04 
- **Loss:** ~$25M 
- **Post-mortem:** [Link to post-mortem](https://dforce.network/blog/post-mortem-analysis-of-lendfme-incident) 
- **Name:** Custom DEX Aggregator Emitted Wrong Token Address
- **Date:** 2022-07 
- **Loss:** ~$90,000 
- **Post-mortem:** [Link to post-mortem](https://immunefi.com)
  
## 📚 Further Reading

- [SWC-136: Incorrect Event Parameters](https://swcregistry.io/docs/SWC-136/) 
- [Solidity Docs – Events](https://docs.soliditylang.org/en/latest/contracts.html#events) 
- [CWE-778: Insufficient Logging](https://cwe.mitre.org/data/definitions/778.html) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Incorrect Event Emissions
severity: L
score:
impact: 3   
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 3.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Off-chain systems may miscalculate rewards, votes, or balances
- **Exploitability**: Can be intentionally used by attacker contracts to confuse tracking
- **Reachability**: Found in any contract with event-based logic
- **Complexity**: Simple mistake; easy to introduce
- **Detectability**: Logs are verifiable; unit tests and audits catch this easily