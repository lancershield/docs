# Block timestamp

```YAML
id: TBA
title: Block timestamp
baseSeverity: H
category: time-manipulation
language: solidity
blockchain: [ethereum]
impact: Premature access, unfair advantages, or delayed execution
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-347
swc: SWC-116
```

## 📝 Description

- The Solidity global variable block.timestamp provides the timestamp of the current block and is often used for time-sensitive logic in smart contracts (e.g., unlock times, auction deadlines, or rate-limiting).
- However, miners or validators can slightly manipulate the value of block.timestamp, typically by up to ±15 seconds, within consensus rules.
- If this variable is used without caution, especially in exact equality comparisons (==) or critical access control (e.g., vesting, withdrawal timing), it may allow:
- Premature access to locked tokens
- Unfair auction sniping
- Denial-of-service by intentionally stalling the time-triggered logic

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Timelock {
    uint256 public unlockTime;
    address public owner;

    constructor(uint256 _unlockTime) {
        unlockTime = _unlockTime;
        owner = msg.sender;
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        require(block.timestamp == unlockTime, "Not time yet"); // ❌ exact match vulnerable
        // send funds
    }
}
```

## 🧪 Exploit Scenario

1. A user sets unlockTime to a specific timestamp.
2. A miner produces a block just before or just after the exact timestamp.
3. The withdrawal fails because block.timestamp never matches unlockTime exactly.
4. Funds become permanently locked or delayed.
5. In MEV-sensitive systems, miners could push timestamps forward slightly to gain advantage in last-move games.

**Assumptions:**

- Contract depends on block.timestamp equality or close range for logic gating.
- No buffer or time window is applied.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeTimelock {
    uint256 public unlockTime;
    address public owner;

    constructor(uint256 _unlockTime) {
        unlockTime = _unlockTime;
        owner = msg.sender;
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        require(block.timestamp >= unlockTime, "Too early"); // ✅ safer check
        // send funds
    }
}
```
## 🧭 Contextual Severity

```yaml
- context: "Timestamp used for randomness or critical access logic"
  severity: H
  reasoning: "Miners can abuse the value to manipulate results or extract value."
- context: "Unlocks and claims that depend on timestamp with no margin"
  severity: M
  reasoning: "Early access or front-running is possible within ±15 seconds."
- context: "Timestamp used for non-critical display or logging"
  severity: I
  reasoning: "Informational usage only; no security implications."
```

## 🛡️ Prevention

### Primary Defenses

- Replace block.timestamp == T with block.timestamp >= T.
- Allow time buffers for execution windows.

### Additional Safeguards

- Use block numbers (block.number) for delay logic where suitable.
- For critical timing in MEV-sensitive apps (e.g., games, auctions), combine block.timestamp with trusted randomness (e.g., Chainlink VRF).

### Detection Methods

- Search for block.timestamp == or block.timestamp < in permission logic.
- Audit vesting, lockup, and auction contracts for time-based access logic.
- Tools: Slither (timestamp-dependence), MythX, custom lint rules

## 🕰️ Historical Exploits

- **Name:** Fomo3D Timestamp Sniping 
- **Date:** 2018 
- **Loss:** 10,469 ETH 
- **Post-mortem:** [Link to post-mortem](https://medium.com/rektify-ai/bad-randomness-in-solidity-8b0e4a393858) 

## 📚 Further Reading

- [SWC-116: Timestamp Dependence](https://swcregistry.io/docs/SWC-116/) 
- [Solidity Security Considerations – Timestamp](https://docs.soliditylang.org/en/latest/security-considerations.html#timestamp-dependence)
- [Trail of Bits: Time-Based Attacks in Ethereum](https://blog.trailofbits.com/) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Block timestamp
severity: H
score:
impact: 3 
exploitability: 3 
reachability: 4  
complexity: 1   
detectability: 5  
finalScore: 3.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Logic relying on time may fail or behave incorrectly.
- **Exploitability**: Timestamp can be influenced within consensus range.
- **Reachability**: Very common in lockups, auctions, and vesting contracts.
- **Complexity**: Simple mistake; typically results from misunderstanding time variance.
- **Detectability**: Obvious to static analyzers like Slither and MythX.
