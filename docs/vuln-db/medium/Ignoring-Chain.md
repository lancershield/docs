# Ignoring Chain Reorgs in Non-Critical Logic

```YAML

id: TBA
title: Ignoring Chain Reorgs in Non-Critical Logic 
severity: M
category: chain-reorg
language: solidity
blockchain: [ethereum]
impact: Off-chain systems or app logic may reflect false data after reorg
status: draft
complexity: medium
attack_vector: blockchain
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-668
swc: SWC-136

```

## 📝 Description

- Ignoring chain reorgs in non-critical logic refers to using block-level values (like `block.number`, `block.timestamp`, `blockhash`, etc.) for application logic, indexing, or external signaling, without considering that these values may be invalidated during reorganization of the blockchain (i.e., a short-range fork).
- While not typically exploitable for direct financial gain, this can:
- Break off-chain synchronization (The Graph, dashboards, monitoring),
- Introduce inconsistencies in state-dependent logic (e.g., voting, claiming, reputation),
- Allow subtle frontrunning or race-condition-like outcomes in dApps relying on reorg-unsafe assumptions.

## 🚨 Vulnerable Code

```solidity
contract ReorgVulnerable {
    event VoteCasted(address indexed voter, uint256 indexed blockNumber);

    function vote() external {
        // ❌ Assumes current block number is final
        emit VoteCasted(msg.sender, block.number);
    }
}

```

## 🧪 Exploit Scenario

Step-by-step impact:

1. A user casts a vote in vote() and off-chain indexer stores this at block.number = 17500000.
2. A short-range reorg occurs, and the new block 17500000 does not contain this vote.
3. On-chain state is valid, but off-chain systems falsely believe the vote occurred.
4. Consensus is broken between on-chain truth and off-chain records.

**Assumptions:**

- Application logic depends on the finality of block.number, block.timestamp, or blockhash.
- No reorg-aware or finality-delayed reconciliation mechanism is in place.

## ✅ Fixed Code

```solidity

contract ReorgSafe {
    event VoteCasted(address indexed voter);

    function vote() external {
        // ✅ Do not emit block.number-dependent data unless justified
        emit VoteCasted(msg.sender);
    }
}
```


## 🛡️ Prevention

### Primary Defenses

- Avoid exposing or relying on block.number, block.timestamp, or blockhash unless essential.
- If exposed, document that data may be unstable until finality (6–15 blocks depending on chain).
- Defer index-sensitive logic to off-chain services with reorg tracking.

### Additional Safeguards

- Use finalized or checkpointed data in L2s or PoS chains.
- Emit reorg-stable identifiers (e.g., tx.origin, msg.sender, event IDs) instead of chain data.
- In off-chain infra, delay processing or confirmation until after finality threshold.

### Detection Methods

- Slither: block-timestamp-dependency, block-number-reliance, chain-dependent-logic detectors.
- Manual audit of functions using block.number, block.timestamp, blockhash, etc.
- Integration testing with forking tools to simulate reorgs.

## 🕰️ Historical Incidents

- **Name:** Graph Protocol Reorg Indexing Gaps 
- **Date:** 2021 
- **Impact:** Misaligned event indexing due to short reorg
-  **Post-mortem:** [Link](https://thegraph.com/docs/faq#what-is-a-reorg) 
  


## 📚 Further Reading

- [SWC-136: Unexpected Behavior due to Environmental Assumptions](https://swcregistry.io/docs/SWC-136) 
- [Solidity Docs – Environment Variables](https://docs.soliditylang.org/en/latest/units-and-global-variables.html)
- [The Graph Docs – Dealing with Reorgs](https://thegraph.com/docs/en/developer/definitions/#reorgs) 
- [Chain Reorgs – How to Handle Them](https://ethereum.org/en/developers/docs/consensus-mechanisms/pow/#chain-reorgs) 
  

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Ignoring Chain Reorgs in Non-Critical Logic 
severity: M
score:
impact: 3         
exploitability: 1 
reachability: 5  
complexity: 2     
detectability: 5  
finalScore: 3.1


```


---

## 📄 Justifications & Analysis

- **Impact**: Can cause silent consensus failures between chain and UI/indexer layers.
- **Exploitability**: Usually indirect, but exploitable if relied on for trustless behavior.
- **Reachability**: Found in nearly all protocols using block.number or block.timestamp for logs or timing.
- **Complexity**: Often unintentional and avoidable.
- **Detectability**: High — flagged by most static tools and observable in global variable usage.