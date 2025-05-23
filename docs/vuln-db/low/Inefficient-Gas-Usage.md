# Inefficient Gas Usage

```YAML
id: TBA
title: Inefficient Gas Usage 
severity: L
category: gas-optimization
language: solidity
blockchain: [ethereum, polygon, arbitrum, optimism, bsc]
impact: Increased gas fees, degraded UX, potential out-of-gas reverts
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-710
swc: SWC-130
```

## 📝 Description

- Smart contracts with poor gas efficiency result in higher transaction costs, worse scalability, and reduced protocol usage. While not directly exploitable, gas inefficiencies can lead to:
- Transactions reverting due to out-of-gas errors
- Inability to batch or loop over large data
- Denial-of-service on high-volume operations
- Uncompetitive products due to UX friction
- Common causes of gas inefficiency include:
- Repeated storage reads/writes
- Unoptimized loops
- Usage of require() or revert() with long strings
- Lack of constant packing or immutables

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Inefficient {
    uint256 public count;

    function incrementMany(uint256 times) external {
        for (uint256 i = 0; i < times; ++i) {
            count = count + 1; // ❌ each write costs 20,000 gas
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A contract allows users to perform N iterations per call (e.g., mint, batch register, batch deposit).
2. Each iteration contains a state write or read.
3. Gas costs accumulate rapidly, causing the transaction to revert mid-loop or become unaffordable to users.
4. The attacker can frontrun or grief others by bloating the mempool with high-gas calls.

**Assumptions:**

- Operations are repeated inefficiently in loops or with redundant writes
- No batching or accumulator patterns are used

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract Optimized {
    uint256 public count;

    function incrementMany(uint256 times) external {
        uint256 localCount = count;
        for (uint256 i = 0; i < times; ++i) {
            localCount += 1; // ✅ do math in memory
        }
        count = localCount; // ✅ single SSTORE at the end
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Apply gas best practices:
- Minimize SSTORE, prefer memory/local variables in loops
- Use immutable and constant for known values
- Avoid expensive string require() messages

## Additional Safeguards

- Use optimizer tools like Solidity-coverage, Gas Reporter, or Forge snapshot
- Use unchecked math where overflow is guaranteed impossible
- Batch operations using mapping accumulators instead of arrays

### Detection Methods

- Slither (gas-inefficiencies, unoptimized-loop),
- Foundry gas snapshots.
- Hardhat Gas Reporter.
- MythX

## 🕰️ Historical Exploits

- **Name:** Uniswap V2 Path Encoding Gas Overhead 
- **Date:** 2020 
- **Loss:** N/A (recognized inefficiency fixed in later designs)
- **Post-mortem:** [Link to post-mortem](https://uniswap.org/whitepaper-v3.pdf)

## 📚 Further Reading

- [SWC-130: Gas Consumption](https://swcregistry.io/docs/SWC-130/) 
- [Solidity Gas Optimizations – Ethereum Wiki](https://ethereum.org/en/developers/docs/gas/#gas-optimization) 
- [OpenZeppelin: Gas Optimization Patterns](https://docs.openzeppelin.com/contracts/4.x/api/utils#GasOptimizations)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Inefficient Gas Usage 
severity: L
score:
impact: 2    
exploitability: 1 
reachability: 5   
complexity: 1    
detectability: 5  
finalScore: 2.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Users pay excessive gas or cannot interact with the contract at scale
- **Exploitability**: Not exploitable in the traditional sense
- **Reachability**: Common in unoptimized loops, large-scale batch ops
- **Complexity**: Easily avoided with training and review
- **Detectability**: Tools like Slither and Gas Reporter flag these automatically
