# Gas Inefficiencies 

```YAML
id: TBA
title: Gas Inefficiencies 
baseSeverity: L
category: gas-optimization
language: solidity
blockchain: [ethereum]
impact: Increased gas cost for users and contract operations
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-398
swc: SWC-135
```

## 📝 Description

- Gas inefficiencies refer to non-optimal code patterns in Solidity that unnecessarily increase transaction costs.       
- While they may not introduce direct security risks or logic errors, they degrade user experience, bloat the blockchain, and waste funds over time. 
- Common patterns include repeated state writes, unindexed events, unnecessary storage access, and bad data structures (e.g., dynamic arrays vs. mappings).

## 🚨 Vulnerable Code

```solidity
contract Inefficient {
    uint public total;

    function updateTotal(uint[] calldata nums) external {
        for (uint i = 0; i < nums.length; i++) {
            total += nums[i]; // repeated storage writes
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step inefficiency scenario:

1. A user calls updateTotal() with 100 items...
2. For each iteration, total += nums[i] performs a separate SLOAD and SSTORE...
3. Gas fees are significantly higher than needed, harming user and contract throughput...

**Assumptions:**

- The contract performs heavy operations in loops.
- Storage slots are accessed or written repeatedly without caching.

## ✅ Fixed Code

```solidity

function updateTotal(uint[] calldata nums) external {
    uint _total = total; // load once into memory
    for (uint i = 0; i < nums.length; i++) {
        _total += nums[i];
    }
    total = _total; // store once
}
```
## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: L
  reasoning: "Increases gas cost but does not affect correctness or security directly."
- context: "High-frequency DeFi dApp"
  severity: M
  reasoning: "Significant cumulative gas loss for users and protocol."
- context: "Low-traffic internal tool"
  severity: I
  reasoning: "Minor concern unless scalability is required."
```

## 🛡️ Prevention

### Primary Defenses

- Cache storage variables in memory during loops.
- Use unchecked blocks when overflow checks are unnecessary.
- Avoid redundant computations inside loops.

### Additional Safeguards
- Prefer mappings over arrays for sparse data.
- Batch updates with fewer storage writes.
- Use constant and immutable variables where applicable.

### Detection Methods

- Slither: gas-cost, uninitialized-state, inefficient-loop detectors.
- Manual audit using gas profiling tools like Remix or Tenderly.
-Optimizer analysis via solc --optimize and bytecode diffing.

## 🕰️ Historical Exploits

- **Name:** Gas-Inefficient Patterns in Deployed Contracts
- **Date:** 2022 
- **Loss:** Widespread inefficiencies in contract executions 
- **Post-mortem:** [Link to post-mortem](https://link.springer.com/article/10.1007/s11390-021-1674-4)
  
## 📚 Further Reading

- [SWC-135: Code With No Effects](https://swcregistry.io/docs/SWC-135) 
- [Gas Optimization in Ethereum Smart Contracts: 10 Best Practices – CertiK](https://www.certik.com/resources/blog/gas-optimization-in-ethereum-smart-contracts-10-best-practices) 
- [Gas Optimization in Ethereum Smart Contracts: Best Practices – Vibranium Audits](https://www.vibraniumaudits.com/post/gas-optimization-in-ethereum-smart-contracts-10-best-practices)

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Gas Inefficiencies
severity: L
score:
impact: 2        
exploitability: 1 
reachability: 4   
complexity: 1     
detectability: 5  
finalScore: 2.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Leads to high transaction costs and inefficiency, but no loss of funds or functionality.
- **Exploitability**: Cannot be exploited in the traditional sense; rather, it causes passive economic degradation.
- **Reachability**: Affects regularly used public/external functions in most contracts.
- **Complexity**: No effort required by the user; impact is implicit in normal use.
- **Detectability**: Readily caught by Slither, Remix, and manual gas profiling.
