# Timestamp Dependency 

```YAML
id: TBA
title: Timestamp Dependency in Critical Logic
severity: M
category: time-manipulation
language: solidity
blockchain: [ethereum]
impact: Time-based logic manipulation by miners or attackers
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-829
swc: SWC-116
```

## 📝 Description

- Timestamp dependency refers to relying on `block.timestamp` (or `now` in older Solidity versions) for critical application logic such as randomness, fund release, or protocol decision-making. 
- Since miners can manipulate timestamps slightly within consensus-accepted bounds, contracts that trust them too much are vulnerable to front-running, delayed execution, or block reordering-based exploits.

## 🚨 Vulnerable Code

```solidity
contract TimeLottery {
    uint public lastDraw;

    function canDraw() public view returns (bool) {
        return block.timestamp >= lastDraw + 1 days;
    }

    function drawWinner() public {
        require(canDraw(), "Too early");
        lastDraw = block.timestamp;
        // Pick winner logic...
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Miner or attacker waits until block.timestamp is just below the required threshold.
2. They include a transaction in their block with a manually adjusted block.timestamp.
3. This enables them to trigger a function earlier than expected, front-run others, or manipulate timing-sensitive logic (e.g., drawWinner, unlock, or mint).
4. This advantage can lead to unfair rewards or predictable execution.

**Assumptions:**

- The contract uses block.timestamp directly for control flow.
- The timing of function execution is economically significant (e.g., lotteries, auctions, staking).

## ✅ Fixed Code

```solidity

function drawWinner() public {
    require(block.timestamp >= lastDraw + 1 days, "Too early");

    // Enforce a minimum block delay to reduce manipulation impact
    require(block.number > lastDrawBlock + 5, "Wait more blocks");

    lastDraw = block.timestamp;
    lastDrawBlock = block.number;
    // Secure winner logic...
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid using block.timestamp for randomness or financial logic.
- Use block numbers instead of timestamps when measuring time duration (e.g., lock periods).
- Implement multi-block thresholds (e.g., block.number > N) to reduce miner influence.

### Additional Safeguards

- Combine timestamps with other entropy sources if needed.
- Use off-chain triggers (e.g., Chainlink Keepers) for timed events.
- Avoid assuming block.timestamp is perfectly accurate or unmanipulable.

### Detection Methods

- Slither: timestamp-dependency detector.
- Manual inspection of time-based conditionals.
- Symbolic execution to test early/late triggers.


## 🕰️ Historical Exploits

- **Name:** GovernMental DApp 
- **Date:** 2016 
- **Loss:** ~1100 ETH (locked/unfair lottery) 
- **Post-mortem:** [Link to post-mortem](https://www.reddit.com/r/ethereum/comments/4np972/governmental_dapp_scam_or_honeypot/) 
  

## 📚 Further Reading

- [SWC-116: Timestamp Dependency](https://swcregistry.io/docs/SWC-116) 
- [Timestamp Dependency in Smart Contracts – GeeksforGeeks](https://www.geeksforgeeks.org/timestamp-dependency-in-smart-contracts/) 
- [Vulnerability: Timestamp Dependence – OWASP Foundation](https://owasp.org/www-project-smart-contract-top-10/2023/en/src/SC03-timestamp-dependence.html) 
- [Time Dependency in Smart Contracts – ImmuneBytes](https://immunebytes.com/blog/time-dependency-in-smart-contracts/)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Timestamp Dependency in Critical Logic
severity: M
score:
impact: 3         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Can lead to unfair participation, early withdrawals, or skipped protocol logic.
- **Exploitability**: Miners can easily manipulate timestamps within ~15 seconds.
- **Reachability**: Most staking, lottery, and unlock patterns use this logic.
- **Complexity**: Moderate; requires being in control of a block or front-running capability.
- **Detectability**: Commonly flagged in audits and by Slither or MythX.
