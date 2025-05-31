# Access Control Vulnerabilities

``` YAML
id: LS16C
title: Access Control Vulnerabilities
baseSeverity: C
category: access-control
language: solidity
blockchain: [ethereum]
impact: Unauthorized state or asset manipulation
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-284
swc: SWC-105
```

## 📝 Description

- Access control vulnerabilities occur when critical functions—such as fund withdrawals, admin configuration, or token minting—lack proper permission checks. 
- In Ethereum smart contracts, this often results from failing to restrict access with `onlyOwner`, role-based modifiers, or faulty logic in `msg.sender` comparisons. 
- Attackers can exploit these flaws to gain unauthorized control over privileged operations.

## 🚨 Vulnerable Code

```solidity
contract Vault {
    address public admin;
    mapping(address => uint256) public balances;

    constructor() {
        admin = msg.sender;
    }

    function withdrawAll() external {
        payable(msg.sender).transfer(address(this).balance); // No access check
    }

    function setAdmin(address _newAdmin) external {
        admin = _newAdmin; // Anyone can call this
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker calls setAdmin() and sets themselves as the admin.
2. Attacker then calls withdrawAll() and drains the contract balance.
3. No access checks prevent unauthorized access to critical logic.

**Assumptions:**

- No onlyOwner/require(msg.sender == admin) present.
- No role-based access control using AccessControl or similar libraries.

## ✅ Fixed Code

```solidity

contract SecureVault is Ownable {
    constructor() {
        transferOwnership(msg.sender);
    }

    function withdrawAll() external onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }

    function setAdmin(address _newAdmin) external onlyOwner {
        transferOwnership(_newAdmin);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Public access to token mint or logic upgrade"
  severity: C
  reasoning: "Leads to unlimited inflation or takeover of contract logic."
- context: "Access control exists but poorly implemented"
  severity: H
  reasoning: "Modifiers present but ineffective or misused."
- context: "Owner-controlled but no emergency pause or multisig"
  severity: M
  reasoning: "Impact confined to trusted actor misuse, not external threats."
```

## 🛡️ Prevention

### Primary Defenses

- Use OpenZeppelin’s Ownable or AccessControl libraries.
- Enforce onlyOwner or onlyRole modifiers on privileged functions.
- Validate msg.sender against trusted addresses before sensitive actions.

### Additional Safeguards

- Consider multi-sig wallets for admin operations.
- Use timelocks for critical governance updates.
- Monitor on-chain admin function calls.

### Detection Methods

- Slither: access-control, missing-authorization detectors.
- MythX, Mythril: symbolic analysis of control flow and privilege escalations.
- Manual auditing with threat modeling and function-level review.

## 🕰️ Historical Exploits

- **Name:** bZx Protocol Admin Key Compromise 
- **Date:** 2021 
- **Loss:** ~$55 million
- **Post-mortem:** [Link to post-mortem](https://rekt.news/bzx-rekt/) 

## 📚 Further Reading

- [SWC-105: Unprotected Critical Function](https://swcregistry.io/docs/SWC-105)
- [OpenZeppelin: Access Control Best Practices](https://docs.openzeppelin.com/contracts/4.x/access-control)
- [Trail of Bits: Access Control Patterns in Smart Contracts](https://www.trailofbits.com/services/software-assurance/blockchain/) 
- [SigmaPrime: Solidity Security Pitfalls](https://blog.sigmaprime.io/solidity-security.html) 

---

## ✅ Vulnerability Report

```markdown
id: LS16C
title: Access Control Vulnerabilities
severity: C
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.2
```

---

## 📄 Justifications & Analysis

- **Impact**: Loss of control, unauthorized withdrawals, or contract destruction.
- **Exploitability**: One call to an unprotected admin function can cause catastrophic failure.
- **Reachability**: Admin-like functions are usually public or external.
- **Complexity**: Little to no complexity if permission checks are missing.
- **Detectability**: Well-known tools like Slither and Mythril detect most access issues, but subtle logic bugs may evade detection.