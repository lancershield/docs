# Unbounded Proposal Gas

```YAML
id: TBA
title: Unbounded Proposal Gas 
severity: H
category: governance
language: solidity
blockchain: [ethereum, optimism, arbitrum, polygon, avalanche]
impact: Governance proposals cannot execute, halting upgrades or DAO actions
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-400
swc: SWC-128
```

## 📝 Description

- Many DAO and protocol governance systems (e.g., Compound-style, OpenZeppelin Governor, custom DAOs) include proposals that can queue and execute multiple actions in one transaction. 
- If these actions are executed in a single unbounded execute() call, the system is vulnerable to gas griefing or denial-of-service (DoS) due to:
- Proposals consuming more gas than the block gas limit
- Inclusion of external or loop-heavy operations
- Attacker-crafted proposals that deliberately exhaust gas before completing
- If even one proposal becomes unexecutable, it can block the queue, especially in FIFO-style timelocks, causing governance paralysis.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract SimpleGovernor {
    struct Proposal {
        address[] targets;
        bytes[] calldatas;
        bool executed;
    }

    mapping(uint256 => Proposal) public proposals;

    function execute(uint256 proposalId) external {
        Proposal storage p = proposals[proposalId];
        require(!p.executed, "Already executed");

        for (uint256 i = 0; i < p.targets.length; ++i) {
            (bool success, ) = p.targets[i].call(p.calldatas[i]); // ❌ unbounded call
            require(success, "Call failed"); // will revert entire proposal
        }

        p.executed = true;
    }
}
```

## 🧪 Exploit Scenario

1. A malicious user submits a proposal with dozens of batched call() actions.
2. The gas cost of executing all actions exceeds the block gas limit.
3. When anyone attempts to execute(), the transaction fails.
4. Since the proposal cannot be removed, no new proposals can pass through the queue, resulting in governance lockup.

**Assumptions:**

- Proposal batching is allowed
- Execution is all-or-nothing with no fallback path

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract ResilientGovernor {
    struct Proposal {
        address[] targets;
        bytes[] calldatas;
        uint256 executedIndex;
    }

    mapping(uint256 => Proposal) public proposals;

    function execute(uint256 proposalId, uint256 limit) external {
        Proposal storage p = proposals[proposalId];
        require(p.executedIndex < p.targets.length, "Already executed");

        uint256 i = p.executedIndex;
        for (; i < p.targets.length && limit > 0; ++i, --limit) {
            (bool success, ) = p.targets[i].call(p.calldatas[i]); // ⚠️ each call runs individually
            require(success, "Execution failed");
        }

        p.executedIndex = i;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Break large proposals into chunks with per-call limits
- Use multi-step execution patterns with checkpointing

### Additional Safeguards

- Add a maxActionsPerProposal limit during proposal creation
- Simulate gas usage of each proposal before queueing
- Reject proposals that exceed a safe gas limit threshold

### Detection Methods

- Detect unbounded for loops over proposal actions inside execute()
- Look for governance systems with no gas constraints
- Tools: Slither (unbounded-loop, gas-dos), Foundry fuzzing, manual review

## 🕰️ Historical Exploits

- **Name:** Aragon Multicall Overflow Bug 
- **Date:** 2023-03 
- **Loss:** N/A (bug caught during internal simulation)
- **Post-mortem:** [Link to post-mortem](https://blog.aragon.org)
  
## 📚 Further Reading

- [SWC-128: Gas Consumption DoS](https://swcregistry.io/docs/SWC-128/) 
- [OpenZeppelin Governor Docs](https://docs.openzeppelin.com/contracts/4.x/governance) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Unbounded Proposal Gas 
severity: H
score:
impact: 4 
exploitability: 4 
reachability: 4  
complexity: 3   
detectability: 4 
finalScore: 4.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Proposals may fail permanently, blocking protocol progression
- **Exploitability**: Anyone with proposal rights can craft a denial proposal
- **Reachability**: Most governance frameworks allow batched execution
- **Complexity**: Requires understanding gas limits and proposal crafting
- **Detectability**: Readily detected by audit or simulation tooling


