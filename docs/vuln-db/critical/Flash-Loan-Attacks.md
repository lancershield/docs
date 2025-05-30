# Flash Loan Exploit

```YAML
id: TBA
title: Flash Loan Exploit 
baseSeverity: C
category: flash-loan
language: solidity
blockchain: [ethereum]
impact: Arbitrary manipulation of protocol logic and state within a single transaction
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.6.0", "<0.8.21"]
cwe: CWE-346
swc: SWC-108
```

## 📝 Description

- Flash loan attacks occur when a protocol allows a user to borrow large amounts of tokens within a single atomic transaction (a "flash loan") and the protocol being exploited fails to account for the possibility of state manipulation or reentry before loan repayment. 
- This allows an attacker to manipulate pricing, collateral, governance votes, or liquidity pools within the same transaction before repaying the loan—making the attack costless and highly capital-efficient.

## 🚨 Vulnerable Code

```solidity
interface IFlashLoanProvider {
    function flashLoan(uint256 amount) external;
}

contract VulnerableVault {
    uint256 public totalDeposits;
    mapping(address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
        totalDeposits += msg.value;
    }

    function withdraw() external {
        uint256 share = balances[msg.sender] / totalDeposits;
        uint256 reward = address(this).balance * share;
        balances[msg.sender] = 0;
        payable(msg.sender).transfer(reward);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker takes a flash loan for a large amount of ETH.
2. Deposits the ETH into the vulnerable contract.
3. totalDeposits increases sharply, skewing share calculations.
4. Immediately withdraws with an inflated reward value.
5. Repays flash loan, keeping the profit.

**Assumptions:**

- No mechanism to differentiate real deposits vs. atomic inflows.
- Arithmetic uses transient state that can be manipulated within a single transaction.

## ✅ Fixed Code

```solidity

function withdraw() external {
    require(tx.origin == msg.sender, "No contract calls");
    require(balances[msg.sender] > 0, "Nothing to withdraw");

    uint256 userBalance = balances[msg.sender];
    balances[msg.sender] = 0;
    totalDeposits -= userBalance;

    payable(msg.sender).transfer(userBalance);
}
```

## 🧭 Contextual Severity

```yaml
- context: "DeFi protocol using DEX price as collateral input"
  severity: C
  reasoning: "Critical – attacker can drain pool using one-block price swing."
- context: "Protocol using Chainlink or TWAP oracles"
  severity: M
  reasoning: "Risk mitigated, but flash loans may still affect liquidity or timing."
- context: "Testnet or simulation"
  severity: L
  reasoning: "No real loss unless deployed in production with real value."
```

## 🛡️ Prevention

### Primary Defenses

- Do not rely on address(this).balance or totalDeposits during the same transaction.
- Use snapshot-based accounting or block-delayed updates.
- Add flash loan-resistant mechanisms (e.g., accrue rewards over multiple blocks).

### Additional Safeguards

- Use tx.origin to restrict contracts (with caveats).
- Integrate flash loan-aware oracles with TWAPs.
- Introduce rate limits, circuit breakers, or oracle price sanity checks.

### Detection Methods

- Symbolic testing of temporal state assumptions.
- Fuzzing with atomic transaction bundles.
- Manual audit of areas involving price, votes, or collateral checks.


## 🕰️ Historical Exploits

- **Name:** Alpha Homora / Cream Finance Exploit
- **Date:** 2021-02-13 
- **Loss:** Approximately $37.5 million 
- **Post-mortem:** [Link to post-mortem](https://blog.alphaventuredao.io/alpha-homora-v2-post-mortem/) 
- **Name:** C.R.E.A.M. Finance Exploit
- **Date:** 2021-10-27 
- **Loss:** Over $130 million 
- **Post-mortem:** [Link to post-mortem](https://www.merklescience.com/blog/hack-track-analysis-of-c-r-e-a-m-finance-hack) 

## 📚 Further Reading

- [SWC-108: Untrusted Delegate Call](https://swcregistry.io/docs/SWC-108/) 
- [OpenZeppelin: Flash Loans and the Advent of Episodic Finance](https://blog.openzeppelin.com/flash-loans-and-the-advent-of-episodic-finance) 
- [Certora: Proof of Optimality of Balancer V2's Flash Loan Bug](https://medium.com/certora/proof-of-optimality-of-balancer-v2s-flash-loan-bug-2eea00b908e2)  
- [Stader Labs: Understanding Flash Loan Attacks](https://www.staderlabs.com/blogs/staking-basics/flash-loan-attack/) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Flash Loan Exploit
severity: C
score:
impact: 5         
exploitability: 5 
reachability: 4   
complexity: 3     
detectability: 3  
finalScore: 4.5
```

## 📄 Justifications & Analysis

- **Impact**: Can completely drain vaults, manipulate markets, or corrupt governance.
- **Exploitability**: Publicly accessible via free capital from flash loan providers.
- **Reachability**: Most DeFi lending and AMM protocols expose price-sensitive logic.
- **Complexity**: Moderate; needs scripting and test environment simulation.
- **Detectability**: Missed by tools unless temporal flows and state coupling are explicitly modeled.
