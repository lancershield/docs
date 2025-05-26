# Backdoor Functionality

```YAML
id: TBA
title: Backdoor Functionality 
baseSeverity: C
category: access-control
language: solidity
blockchain: [ethereum]
impact: Total contract compromise or asset theft
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-912
swc: SWC-100
```

## 📝 Description

- Backdoor Functionality refers to hidden code paths, functions, or logic embedded within a smart contract that give the contract deployer, admin, or attacker the ability to:
- Drain funds,Arbitrarily manipulate state,Modify balances,Reassign ownership or privileges,without the knowledge or approval of users interacting with the contract.
- Such code may be obfuscated, conditionally triggered, or protected behind misleading modifiers.
- While sometimes inserted intentionally for rug pulls, it can also stem from sloppy privilege management or insecure defaults.

## 🚨 Vulnerable Code

```solidity
contract Token {
    mapping(address => uint256) public balanceOf;
    address public admin;

    constructor() {
        admin = msg.sender;
        balanceOf[admin] = 1_000_000 * 1e18;
    }

    function rugPull() external {
        require(msg.sender == admin, "Not admin");
        payable(admin).transfer(address(this).balance);
    }

    function resetBalance(address user) external {
        if (msg.sender == admin) {
            balanceOf[user] = 0;
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Deployer includes a rugPull() function disguised among other utility functions.
2. Users interact with the token, unaware of the malicious code.
3. At a chosen moment, the deployer calls rugPull() to drain funds and resetBalance() to zero out wallets.
4. Users are left with worthless tokens or empty balances.

## Assumptions:

- Users did not audit or notice the hidden logic.
- Functions are gated behind admin checks or obfuscated names.
- No on-chain governance or revocation mechanisms exist.

## ✅ Fixed Code

```solidity

// No hidden privileged functions
contract Token {
    mapping(address => uint256) public balanceOf;
    uint256 public totalSupply;

    constructor() {
        totalSupply = 1_000_000 * 1e18;
        balanceOf[msg.sender] = totalSupply;
    }

    // No rug pull, no reset logic
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: C
  reasoning: "Violates trust, allows silent theft or sabotage of user funds."
- context: "Privately deployed tool with disclosed emergency controls"
  severity: M
  reasoning: "Still risky but users are aware and consent to the design."
- context: "DAO-managed contract with on-chain audits"
  severity: L
  reasoning: "Unlikely, as transparency and governance limit misuse."
```

## 🛡️ Prevention

### Primary Defenses

- Use role-based access controls with community-governed administration (e.g., OpenZeppelin AccessControl).
- Disallow any functions that arbitrarily modify critical state unless strictly required and governed.
- Eliminate hardcoded privileged roles post-deployment (renounce ownership or use timelocks).

### Additional Safeguards

- Conduct full external audits on production contracts.
- Mandate open-sourcing and reproducible builds.
- Integrate immutable proxies where logic can’t be modified post-deployment.

### Detection Methods

- Slither: dangerous-functions, admin-access, arbitrary-state-write detectors.
- Mythril/MythX: symbolic execution highlighting unchecked control paths.
- Manual auditing: particularly functions with onlyOwner, admin, or god prefixes.

## 🕰️ Historical Exploits

- **Name:** Parity Wallet Hack 
- **Date:** 2017 
- **Loss:** Over $30 million worth of Ethereum 
- **Post-mortem:** [Link to post-mortem](https://codeofcode.org/lessons/case-studies-of-real-world-smart-contract-vulnerabilities-and-exploits/)

## 📚 Further Reading

- [SWC-100: Function Default Visibility](https://swcregistry.io/docs/SWC-100/) 
- [OWASP Smart Contract Top 10](https://owasp.org/www-project-smart-contract-top-10/) 
- [GitHub - SunWeb3Sec/DeFiVulnLabs](https://github.com/SunWeb3Sec/DeFiVulnLabs)
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Backdoor Functionality
severity: C
score:
impact: 5         
exploitability: 4 
reachability: 3   
complexity: 2     
detectability: 3  
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Full contract compromise—funds and balances can be drained or modified.
- **Exploitability**: Admin can call functions anytime post-deployment.
- **Reachability**: Not always public, but accessible by attacker/deployer.
- **Complexity**: Simple to write, sometimes disguised under misleading names.
- **Detectability**: Moderate — easier with Slither, hard for casual review or non-dev users.

