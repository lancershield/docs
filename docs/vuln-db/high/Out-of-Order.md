# Out-of-Order Retryable Transactions

```YAML
id: TBA
title: Out-of-Order Retryable Transactions 
severity: H
category: l2-behavior
language: solidity
blockchain: [arbitrum, optimism, ethereum]
impact: Unexpected reentrancy, double-spend, or stale state execution
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-362
swc: SWC-107
```

## 📝 Description

- Some L2 rollups (e.g., Arbitrum and Optimism) support retryable transactions—transactions that fail on first execution but can later be re-executed via user or system calls. 
- However, these retryables can be executed out of order relative to the original L1 trigger, creating dangerous edge cases where:
- Contract assumptions about sequential state updates break
- Re-entrant logic executes using outdated state
- Funds can be double-claimed or state-dependent access controls bypassed
- If contracts assume that retryables will execute immediately after their originating L1 message or that no prior state has changed, they are vulnerable to re-entrancy, stale data, or logic corruption.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract L2BridgeReceiver {
    mapping(address => uint256) public balances;

    function finalizeDeposit(address user, uint256 amount) external {
        require(balances[user] == 0, "Already credited"); // ❌ fragile assumption
        balances[user] = amount;
    }
}
```

## 🧪 Exploit Scenario

1. An L1 deposit triggers a retryable message to L2.
2. That retryable is delayed due to congestion or manual retry.
3. Meanwhile, the user interacts with L2 and withdraws or re-deposits.
4. The retryable is executed later, under invalid assumptions about the user's prior state.
5. This may result in:
6. Duplicate credit
7. Re-entrant balance updates
8. Out-of-order logic execution

**Assumptions:**

- L2 contract logic assumes strict ordering of messages and state updates.
- Retryables are not idempotent or guarded against reentrancy.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeL2BridgeReceiver {
    mapping(address => uint256) public balances;
    mapping(bytes32 => bool) public processed;

    function finalizeDeposit(bytes32 msgId, address user, uint256 amount) external {
        require(!processed[msgId], "Already finalized");  // ✅ idempotent check
        processed[msgId] = true;
        balances[user] += amount;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Never assume L1→L2 or internal retryables will execute in a specific order.
- Protect against duplicate processing using message hashes or IDs.

### Additional Safeguards

- Ensure all retryable-triggered logic is idempotent.
- Use L2 inbox middleware (like Arbitrum’s RetryableTx guard) for deterministic execution.

### Detection Methods

- Search for functions processing cross-chain messages without uniqueness constraints.
- Audit for state assumptions that rely on sequential message processing.
- Tools: Static analyzers, manual review, L2-specific testing frameworks

## 🕰️ Historical Exploits

- **Name:** Offchain Labs Out-of-Order Replay Warning 
- **Date:** 2021 
- **Loss:** None (acknowledged in L2 whitepaper) 
- **Post-mortem:** [Link to post-mortem](https://developer.arbitrum.io/arbos/l1-to-l2-messaging/#retryable-transactions) 

---

## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107/) 
- [Out-of-order retryable transactions – SLITHER-W1093 – DeepSource](https://deepsource.com/directory/slither/issues/SLITHER-W1093) 
- [Detector Documentation – Slither Wiki](https://github.com/crytic/slither/wiki/Detector-Documentation)
- [Smart Contract Vulnerabilities Unveiled: Transaction Ordering Dependence – Coinmonks](https://medium.com/coinmonks/smart-contract-vulnerabilities-unveiled-transaction-ordering-dependence-tod-b13a832be692) 

---
  
## ✅ Vulnerability Report

```markdown
id: TBA
title: Out-of-Order Retryable Transactions 
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 4   
complexity: 3     
detectability: 3 
finalScore: 3.6
```

---

## 📄 Justifications & Analysis

- **Impact**: Replay or out-of-order retryables can break state logic and corrupt accounting.
- **Exploitability**: Attackers may intentionally delay execution or trigger replays.
- **Reachability**: Frequent in L2 bridges, rollup integrations, and automated systems.
- **Complexity**: High if L1–L2 communication isn’t well-understood or documented.
- **Detectability**: Hard to detect without understanding L2 queueing and retry semantics.