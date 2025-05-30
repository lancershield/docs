# Vote Escrow Exploits

```YAML
id: TBA
title: Vote Escrow Exploits 
baseSeverity: H
category: governance
language: solidity
blockchain: [ethereum]
impact: Governance manipulation or protocol hijack
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.5.0", "<0.8.21"]
cwe: CWE-1236
swc: SWC-136
```

## 📝 Description

- Vote Escrow Exploits refer to vulnerabilities in protocols using vote escrowed tokens (e.g., veToken systems like Curve’s veCRV), where voting power is granted in proportion to token locks, often with time-based decay. 
- Poorly designed escrow mechanisms can allow:
- Flash voting using temporary token balance boosts
- Double-voting across multiple snapshots
- Boosting voting weight via strategic token transfer or lock manipulation
- Bypassing decay or abusing delegation features.
- These can lead to manipulated governance outcomes, including malicious proposal passing, reward redirection, or protocol takeovers.

## 🚨 Vulnerable Code

```solidity

mapping(address => uint256) public locked;
mapping(address => uint256) public votePower;

function lockTokens(uint256 amount, uint256 duration) external {
    token.transferFrom(msg.sender, address(this), amount);
    locked[msg.sender] += amount;
    votePower[msg.sender] += amount * duration; // naive boost
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker acquires tokens shortly before a governance snapshot.
2. Calls lockTokens() for max duration to get high voting weight.
3. Votes on a malicious proposal.
4. After vote passes, attacker unlocks tokens or exits via token transfer or manipulation.
5. Community is left with a permanent governance change from a transient attacker.

**Assumptions:**

- Protocol does not prevent flash or short-duration locks.
- Voting snapshot is taken after the lock boost.
- Locking is not enforced to match snapshot voting behavior.

## ✅ Fixed Code

```solidity

function lockTokens(uint256 amount, uint256 duration) external {
    require(duration >= 1 weeks, "Minimum lock");
    require(duration <= MAX_DURATION, "Exceeds max");

    token.transferFrom(msg.sender, address(this), amount);
    uint256 unlockTime = block.timestamp + duration;

    // vote power = f(amount, unlockTime)
    locked[msg.sender] = amount;
    votePower[msg.sender] = amount * (unlockTime - block.timestamp) / MAX_DURATION;
}
```
## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: H
  reasoning: "Unrestricted voting abuse can shift protocol control unfairly."
- context: "Tightly controlled DAO with multisig proposal approvals"
  severity: M
  reasoning: "Requires on-chain voting plus multisig execution, reducing impact."
- context: "Centralized governance with off-chain votes"
  severity: L
  reasoning: "Actual vote execution is off-chain; abuse is informational only."
```

## 🛡️ Prevention

### Primary Defenses

- Use time-decay weighted voting based on actual lock duration (like veCRV).
- Prevent flash-loanable vote weights by enforcing lock duration.
- Use snapshot-based voting aligned with fixed timestamps.

### Additional Safeguards

- Delay proposal execution to detect manipulation.
- Require quorum and time-weighted participation.
- Cap voting influence per address to prevent Sybil-like boosts.

### Detection Methods

- Static audit of vote-power calculation functions.
- Test governance flow against replay, delegation, and snapshot edge cases.
- Fuzz voting scenarios to simulate adversarial behavior.

## 🕰️ Historical Exploits

- **Name:** Yearn vs. Convex in the Curve Wars 
- **Date:** 2021 
- **Loss:** No direct financial loss, but raised concerns over governance centralization 
- **Post-mortem:** [Link to post-mortem](https://www.flovtec.com/post/the-curve-wars-explained)   

## 📚 Further Reading

- [SWC-136: Unexpected Behavior from External Calls – SWC Registry](https://swcregistry.io/docs/SWC-136/) 
- [Beanstalk Governance Hack Analysis – Rekt](https://rekt.news/beanstalk-rekt/)
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Vote Escrow Exploits 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 3   
complexity: 3     
detectability: 3  
```

---

## 📄 Justifications & Analysis

- **Impact**: Enables malicious proposal passing or reward capture.
- **Exploitability**: Medium-high if locks and snapshots are poorly designed.
- **Reachability**: Affects all vote-escrow-based protocols.
- **Complexity**: Requires strategic timing or scripting.
- **Detectability**: Can slip through basic reviews unless simulation or snapshot testing is done.


