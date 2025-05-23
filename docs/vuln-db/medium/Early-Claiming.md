# Early Claiming of Rewards

```YAML
id: TBA
title: Early Claiming of Rewards
severity: M
category: timing
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: Users can bypass intended vesting or reward schedules
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-285: Improper Authorization
swc: SWC-116: Block Values as a Proxy for Time
```

## 📝 Description

- Early Claiming of Rewards" refers to a vulnerability where users are able to claim rewards (tokens, points, or benefits) before the protocol intended, usually by exploiting incorrect or imprecise time validation.
- Common causes: Relying on block numbers instead of timestamps (leading to miner-controlled drift)
- Off-by-one errors in startTime, claimStart, or duration logic
- Skipping precondition checks such as hasStarted, vested() > 0, or claimableTimeReached
- This allows attackers or fast-claiming bots to collect unfair advantages and disrupt economic assumptions such as vesting schedules, loyalty rewards, or linear distributions.

## 🚨 Vulnerable Code

```solidity

uint256 public start;
uint256 public duration;
mapping(address => bool) public hasClaimed;

function claim() external {
    require(!hasClaimed[msg.sender], "Already claimed");
    require(block.number >= start, "Too early"); // ❌ susceptible to manipulation

    hasClaimed[msg.sender] = true;
    rewardToken.transfer(msg.sender, 100 * 1e18);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Protocol schedules reward claims to start at a specific time, but uses block.number for validation.
2. Miner or fast-claiming user submits a transaction in a borderline block (e.g., slightly before intended time).
3. The block gets mined faster than expected, and early claim succeeds.
4. Attacker collects reward and prevents others from receiving theirs in cases of single-claim limits.
5. Vesting or fairness assumptions are broken; trust in distribution is reduced.

**Assumptions:**

- Claim conditions rely on inaccurate time metrics.
- Claimable state is not locked by proper schedule validation.
- Public claim() function has no user-specific vesting rules.

## ✅ Fixed Code

```solidity

uint256 public claimStart;
uint256 public duration;
mapping(address => bool) public hasClaimed;

function claim() external {
    require(!hasClaimed[msg.sender], "Already claimed");
    require(block.timestamp >= claimStart, "Claim period not started");

    hasClaimed[msg.sender] = true;
    rewardToken.transfer(msg.sender, 100 * 1e18);
}
```

## 🛡️ Prevention

### Primary Defenses

- Use block.timestamp, not block.number, for all time checks.
- Enforce per-user schedules via Merkle roots or mapping claims to vesting schedules.
- Ensure vesting math has no off-by-one errors or premature unlocks.

### Additional Safeguards

- Add hasStarted() checks for phases like whitelist, claim, vesting, etc.
- Require role-based trigger of distribution windows (e.g., DAO-controlled activation).
- Simulate reward functions during audits to validate claim logic.

### Detection Methods

- Static analysis for block.number used in timing conditions.
- Unit tests with boundary conditions and early block scenarios.
- Tools: Slither (timestamp-dependency), MythX, Foundry property tests

## 🕰️ Historical Exploits

- **Name:** Ampleforth Geyser Early Rewards 
- **Date:** 2020-10 
- **Loss:** ~4.9% 
- **Post-mortem:** [Link to post-mortem](https://medium.com/ampleforth/) 

## 📚 Further Reading

- [SWC-116: Block Values as a Proxy for Time](https://swcregistry.io/docs/SWC-116) 
- [CWE-285: Improper Authorization](https://cwe.mitre.org/data/definitions/285.html) 
- [Solidity Security – Use block.timestamp for Timing](https://docs.soliditylang.org/en/v0.8.25/security-considerations.html#timestamp-dependencies) 
  
---
 
## ✅ Vulnerability Report

```markdown
id: TBA
title: Early Claiming of Rewards
severity: M
score:
impact: 3 
exploitability: 4  
reachability: 5  
complexity: 2   
detectability: 3 
finalScore: 3.45
```

---

## 📄 Justifications & Analysis

- **Impact**: Allows attackers to gain disproportionate control or rewards, diluting fairness and undermining trust in reward or governance systems.
- **Exploitability**: Easy to exploit using standard scripts, wallet generators, or Sybil farms; no sophisticated tooling required.
- **Reachability**: Common in DeFi airdrops, NFT mints, DAO voting, and per-wallet reward logic.
- **Complexity**: Requires only basic scripting skills to generate EOAs and automate transactions.
- **Detectability**: Detectable via clustering heuristics, behavioral analytics, or static analysis for lack of identity gating.
