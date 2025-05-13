# Flash Loan Attacks

```YAML
id: TBA
title: Flash Loan Exploit on Atomic Lending Operations
severity: C
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

- Attacker takes a flash loan for a large amount of ETH.
- Deposits the ETH into the vulnerable contract.
- totalDeposits increases sharply, skewing share calculations.
- Immediately withdraws with an inflated reward value.
- Repays flash loan, keeping the profit.

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

- **Name:** bZx Protocol Flash Loan Attack
- **Date:** 2020-02-15 
- **Loss:** ~$350K 
- **Post-mortem:** [Link to post-mortem](https://bzx.network/blog/postmortem-incident-feb-15th) - 
 

- **Name:** Alpha Homora / Cream Exploit 
- **Date:** 2021-02-13 
- **Loss:** ~$37.5M 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/alpha-homora-rekt/) 


## 📚 Further Reading

- [SWC-108: Untrusted Delegate Call or Flash Loan](https://swcregistry.io/docs/SWC-108) 
- [OpenZeppelin – Flash Loan Security](https://blog.openzeppelin.com/defending-defi-flash-loan-attacks/) 
- [Certora – Flash Loan Vulnerability Modeling](https://blog.certora.com/why-do-flash-loans-keep-breaking-defi-22f4d93b6299) 

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Flash Loan Attacks
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
