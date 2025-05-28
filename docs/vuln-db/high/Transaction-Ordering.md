# Transaction Ordering Dependence

```YAML
id: TBA
title: Transaction Ordering Dependence 
baseSeverity: H
category: ordering
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: User transactions can be front-run, re-ordered, or invalidated
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-362
swc: SWC-114
```

## 📝 Description

- Transaction Ordering Dependence (TOD), also known as "front-running," is a class of vulnerabilities where the order of transactions in a block affects contract behavior, enabling attackers to profit by observing and reordering pending transactions.
- This typically arises when:
- A contract has time-sensitive or conditional logic (e.g., first-come, first-served)
- There is no mechanism to lock, commit, or protect sensitive functions
- External observers (e.g., miners, MEV bots) can simulate or monitor the mempool
- Affected patterns include:
- Auctions or reward claims favoring the first caller
- Commit-reveal games with no anti-front-running protection
- Functions that rely on temporary global state like block.timestamp or tx.origin

## 🚨 Vulnerable Code

```solidity

bool public claimed;

function claim() external {
    require(!claimed, "Already claimed");
    claimed = true;
    rewardToken.transfer(msg.sender, 100 * 1e18);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A protocol implements a one-time claim() function that rewards the first caller.
2. Alice prepares and signs her transaction, which enters the mempool.
3. A bot monitors mempool and detects Alice’s intent.
4. The bot submits a transaction with a higher gas price, front-running Alice’s tx.
5. The contract sets claimed = true in the bot’s tx; Alice’s transaction fails.
6. The bot gets the full reward, and Alice receives nothing but pays gas.

**Assumptions:**

- No commit-reveal or proof-of-intent mechanism.
- Global state is changed in a way that depends on tx ordering.
- Publicly accessible functions without per-user protection.

## ✅ Fixed Code

```solidity

mapping(address => bool) public hasClaimed;

function claim() external {
    require(!hasClaimed[msg.sender], "Already claimed");
    hasClaimed[msg.sender] = true;
    rewardToken.transfer(msg.sender, 100 * 1e18);
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Outcome is influenced by mempool visibility and ordering."
- context: "Public DeFi protocol with shared state logic"
  severity: C
  reasoning: "Highly exploitable via MEV searchers or validator frontrunning."
- context: "Private contracts or batch-queued execution"
  severity: L
  reasoning: "Limited exposure due to controlled execution context."
```

## 🛡️ Prevention

### Primary Defenses

- Use commit-reveal schemes for time-sensitive logic.
- Implement per-user claim tracking instead of global flags.
- Delay effect execution using timelocks or randomization.

### Additional Safeguards

- Leverage priority queues instead of first tx wins.
- Use sealed-bid models for auctions or lotteries.
- Integrate with MEV protection tools (e.g., Flashbots RPC)

### Detection Methods

- Static analysis for global state flags modified in user-facing functions.
- Simulation tools for reordering attack viability.
- Tools: Slither (tx-order-dependence), MythX, Tenderly simulation

## 🕰️ Historical Exploits

- **Name:** Bancor First-Stake Race 
- **Date:** 2020 
- **Loss:** N/A 
- **Post-mortem:** [Link to post-mortem](https://blog.bancor.network/) 
  
## 📚 Further Reading

- [SWC-114: Transaction Order Dependence](https://swcregistry.io/docs/SWC-114) 
- [CWE-362: Shared Resource Race Condition](https://cwe.mitre.org/data/definitions/362.html) 
- [Flashbots – Introduction to MEV](https://docs.flashbots.net/)
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Transaction Ordering Dependence 
severity: H
score:
impact: 4   
exploitability: 4  
reachability: 5   
complexity: 2  
detectability: 3  
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Allows loss of user rewards or unfair advantages in race conditions.
- **Exploitability**: Easily triggered via gas manipulation or Flashbots bundles.
- **Reachability**: Present in global claim, game, and auction patterns.
- **Complexity**: Straightforward for anyone familiar with tx reordering and gas bidding.
- **Detectability**: Can be caught with Slither and mempool simulation tools.
