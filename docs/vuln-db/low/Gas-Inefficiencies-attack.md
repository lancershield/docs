# Gas Inefficiencies 


```YAML
id: TBA
title: Gas Inefficiencies via Suboptimal Solidity Patterns
severity: L
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


  - **Name:** ZRX Token Transfer Loop Bug 
  - **Date:** 2018-08-19 
  - **Loss:** N/A (No hack, but excessive gas burned)  
  - **Post-mortem:** [Link to post-mortem](https://medium.com/0xproject/bug-disclosure-token-transfer-loop-bug-cf63e3f2188e) 

## 📚 Further Reading


- [SWC-135: Code With No Effects](https://swcregistry.io/docs/SWC-135) 
-  [Solidity Docs – Gas Optimization Tips](https://docs.soliditylang.org/en/latest/internals/optimizing-gas-costs.html) 
-  [Slither – Gas Optimization Detectors](https://github.com/crytic/slither)


 ---

## ✅ Vulnerability Report 


```markdown
id: vuln__gas_inefficiencies
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

-**Impact**: Leads to high transaction costs and inefficiency, but no loss of funds or functionality.

-**Exploitability**: Cannot be exploited in the traditional sense; rather, it causes passive economic degradation.

-**Reachability**: Affects regularly used public/external functions in most contracts.

-**Complexity**: No effort required by the user; impact is implicit in normal use.

-**Detectability**: Readily caught by Slither, Remix, and manual gas profiling.
