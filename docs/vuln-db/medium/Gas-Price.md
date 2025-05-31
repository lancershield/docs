# Gas Price Dependent Logic

```YAML
id: LS28M
title: Gas Price Dependent Logic
baseSeverity: M
category: logic
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Logic or access decisions manipulated via custom gas prices
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-670
swc: SWC-128
```

## 📝 Description

- Gas Price Dependent Logic occurs when smart contract functions include checks or conditions based on tx.gasprice or block.basefee. 
- Since gas prices are user-controlled (or miner-influenced), they are not a reliable source of truth for:
- Access control
- Randomness or timing
- Transaction prioritization
- This opens the door to manipulation, denial-of-service, or privilege escalation attacks—especially on L1 chains or during gas auctions.

## 🚨 Vulnerable Code

```solidity

function claim() external {
    require(tx.gasprice < 50 gwei, "Gas price too high"); // ❌ logic based on manipulable value
    claimed[msg.sender] = true;
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A contract uses tx.gasprice to reject or prioritize certain transactions.
2. The attacker sets a very low gas price (e.g., 1 gwei) to pass a claim function check.
3. Meanwhile, honest users using normal gas prices (e.g., 30–50 gwei) are rejected.
4. In reverse, a privileged path may be denied to all users by spamming with high-gas transactions.

**Assumptions:**

- The contract uses tx.gasprice or block.basefee in conditionals.
- Attackers can submit transactions with arbitrary gas prices or bribe miners.
- Honest users cannot predict or simulate consistent gas price behavior.

## ✅ Fixed Code

```solidity

function claim() external {
    require(!claimed[msg.sender], "Already claimed");
    claimed[msg.sender] = true;
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "General misuse leads to unpredictable behavior but limited direct loss."
- context: "Protocols relying on gas-based randomness"
  severity: H
  reasoning: "Randomness becomes manipulable, leading to jackpot exploits or loss of fairness."
- context: "Logging or statistical use only"
  severity: L
  reasoning: "Minimal impact if tx.gasprice is not used in decision-making logic."
```

## 🛡️ Prevention

### Primary Defenses

- Never use tx.gasprice or block.basefee for control flow or eligibility logic.
- Use timestamps (block.timestamp), block numbers, or verifiable oracles for decisions.
- For fairness or randomness, use Chainlink VRF or RANDAO.

### Additional Safeguards

- Log gas prices for transparency if needed, but don’t act on them.
- Use difficulty-adjusted block values only in PoW chains with caution.
- Validate input fairness off-chain, not in contract logic.

### Detection Methods

- Static analysis for tx.gasprice, block.basefee, or related conditionals.
- Manual audit of require/assert statements using gas-dependent values.
- Tools: Slither (gas-price-check), MythX symbolic analysis

## 🕰️ Historical Exploits

- **Name:** GasToken Abuse via Refund Mechanism 
- **Date:** 2018 
- **Loss:** ~$100,000
- **Post-mortem:** [Link to post-mortem](https://gastoken.io/) 

## 📚 Further Reading

- [SWC-128: DoS With Block Gas Limit](https://swcregistry.io/docs/SWC-128) 
- [CWE-670: Always-Incorrect Control Flow](https://cwe.mitre.org/data/definitions/670.html) 
- [Chainlink VRF](https://docs.chain.link/docs/vrf/v2/introduction/) 
- [Ethereum Docs – tx.gasprice Warning](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#gas-and-transactions)

--- 

## ✅ Vulnerability Report

```markdown
id: LS28M
title: Gas Price Dependent Logic
severity: M
score:
impact: 3  
exploitability: 4  
reachability: 3   
complexity: 2  
detectability: 4   
finalScore: 3.25
```

---

## 📄 Justifications & Analysis

- **Impact**: May block users or unfairly advantage attackers based on gas price tricks.
- **Exploitability**: Simple for any attacker to manipulate if relying on tx.gasprice.
- **Reachability**: Found in lotteries, claims, raffles, and conditional access contracts.
- **Complexity**: Low—submit low-gas tx to win access or block others.
- **Detectability**: Static analyzers catch this via known patterns.

