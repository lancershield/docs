# Bribery Through Vote Delegation

```YAML
id: TBA
title: Bribery Through Vote Delegation 
severity: M
category: governance
language: solidity
blockchain: [ethereum]
impact: Protocol decisions can be skewed by economic incentives unrelated to long-term interests
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.6.0", "<latest"]
cwe: CWE-732
swc: SWC-133
```

## 📝 Description

- Bribery through vote delegation occurs when token holders are incentivized to delegate their governance votes in exchange for bribes, payments, or yield, enabling:
- Governance capture by a single whale or bribing contract Misaligned decision-making, where votes representshort-term gain rather than community values,Protocol behavior that favors the briber (e.g., fee allocation, token emissions, treasury grants).
- This is particularly dangerous in protocols using vote delegation (e.g., Compound Governor Bravo, ERC20Votes, or veToken models) without enforcing voting power restrictions or quorum diversity.

## 🚨 Vulnerable Code

```solidity
// ❌ No restrictions or disclosure for delegated vote usage
function delegate(address delegatee) public {
    _delegate(msg.sender, delegatee); // Can be bribed off-chain
}
```

## 🧪 Exploit Scenario

Step-by-step bribery:

1. A governance proposal is introduced to change the reward distribution in a DEX.
2. A whale or bribing platform (e.g., vote market) offers users yield in return for delegating votes.
3. Thousands of users delegate to the bribing contract, increasing its vote weight.
4. The bribing contract uses the delegated votes to pass a self-beneficial proposal — e.g., boosting emissions to its LP.
5. Protocol funds and rewards are now misallocated, hurting non-participants and damaging governance integrity.

**Assumptions:**

- Bribe markets like Votemarket or Hidden Hand allow coordinated vote selling.
- Protocol fails to detect or limit vote concentration via off-chain delegation incentives.

## ✅ Fixed Code

```solidity
// ✅ Limit max delegation per address (soft control)
modifier enforceDecentralization(address delegatee) {
    require(delegateWeight[delegatee] < MAX_DELEGATION_CAP, "Delegate exceeds cap");
    _;
}

// ✅ Track source of delegation for transparency
mapping(address => address) public delegatedFrom;

function delegate(address delegatee) public {
    delegatedFrom[msg.sender] = delegatee;
    _delegate(msg.sender, delegatee);
}
```

## 🛡️ Prevention

### Primary Defenses

- Monitor for rapid vote delegation spikes before proposals.
- Encourage vesting-based governance (e.g., veToken models with lock-in).
- Implement reputation penalties or rate limits for high-flux delegators.

### Additional Safeguards

- Track on-chain vote delegation flows and audit proposal influence concentration.
- snapshot-based voting models to reduce real-time delegation manipulation.
- Apply multi-sig or council overrides in early-stage governance for safety.

### Detection Methods

- Slither: Detect rapid delegation state changes (delegate-churn, governance-bribery-pattern).
- Analyze governance participation metrics for voter clustering or reuse.
- Monitor on-chain bribe payout contracts and linked wallets.

## 🕰️ Historical Exploits

- **Name:** Arbitrum DAO Governance Incident 
- **Date:** April 2025 
- **Loss:** Undisclosed influence over governance decisions 
- **Post-mortem:** [Link to post-mortem](https://www.blocmates.com/news-posts/arbitrum-dao-governance-faces-scrutiny-following-vote-delegation-via-lobbyfi)


## 📚 Further Reading

- [SWC-133: Governance Manipulation](https://swcregistry.io/docs/SWC-133) 
- [Compound Governor Bravo Docs](https://docs.compound.finance/v2/governance/) 
- [A Game-Theoretic Analysis of Delegation Incentives in Blockchain Governance – IOHK](https://iohk.io/en/research/library/papers/a-game-theoretic-analysis-of-delegation-incentives-in-blockchain-governance/)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Bribery Through Vote Delegation 
severity: M
score:
impact: 4         
exploitability: 3 
reachability: 5   
complexity: 3     
detectability: 4  
finalScore: 3.85

```

---

## 📄 Justifications & Analysis

- **Impact**: High — governance distortion can redirect protocol funds, emissions, or upgrades.
- **Exploitability**: Moderate — requires incentive system and coordination but no smart contract flaw.
- **Reachability**: Very high — all vote-delegation systems are theoretically vulnerable.
- **Complexity**: Moderate — bribery platforms and vote tracking make this easy to scale.
- **Detectability**: High — patterns emerge in on-chain voting data and bribe payouts.