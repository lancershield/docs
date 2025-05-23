# Missing Input Validation 

```YAML
id: TBA
title: Missing Input Validation 
severity: H
category: input-validation
language: solidity
blockchain: [ethereum, polygon, optimism, arbitrum, bsc]
impact: Unauthorized access, corrupted state, or protocol malfunction
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-20
swc: SWC-135
```

## 📝 Description

- Missing input validation occurs when smart contract functions fail to check inputs such as:
- Zero addresses
- Invalid array lengths
- Out-of-bounds indexes
- Malformed token amounts
- Duplicate or unverified external calls
- This can result in state corruption, unauthorized behavior, and unintentional asset loss, especially in permissioned systems or DeFi protocols that rely on structured inputs for asset transfer, role assignment, or governance logic.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Registry {
    mapping(address => bool) public isRegistered;

    function register(address user) external {
        isRegistered[user] = true; // ❌ no check for zero address
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A malicious actor calls register(address(0)).
2. Any logic using isRegistered[user] blindly trusts that address(0) is valid.
3. If future logic treats address(0) as a special case (e.g. withdrawal target, governance override), the attacker can abuse this to gain control, bypass checks, or trap funds.

**Assumptions:**

- No checks for zero address, bad formatting, or structural input constraints.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract Registry {
    mapping(address => bool) public isRegistered;

    function register(address user) external {
        require(user != address(0), "Invalid address");
        isRegistered[user] = true;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always validate,address inputs ≠ address(0),uint amounts > 0 (when relevant)
- Array lengths > 0 and consistent
- No duplicated keys (if relevant)
- Validate data in modifiers or dedicated internal checks

### Additional Safeguards

- Use input schemas or structs where appropriate
- Assert preconditions before modifying state

### Detection Methods

- Audit all public/external functions for require() or validation guards
- Tools: Slither (missing-zero-check, unchecked-call-arg), MythX, custom assertions

## 🕰️ Historical Exploits

- **Name:** Poly Network Cross-Chain Exploit 
- **Date:** 2021-08 
- **Loss:** ~$600 million stolen due to missing input validation in cross-chain message handling 
- **Post-mortem:** [Link to post-mortem](https://coinmetro.com/learning-lab/smart-contract-vulnerabilities-case-studies) 
- **Name:** Wormhole Bridge Signature Forgery 
- **Date:** 2022-02 
- **Loss:** ~$324 million minted via forged guardian signatures due to lack of input validation 
- **Post-mortem:** [Link to post-mortem](https://www.hackerone.com/blog/smart-contracts-common-vulnerabilities-and-real-world-cases) 

## 📚 Further Reading

- [SWC-135: Incorrect Event or Argument Encoding](https://swcregistry.io/docs/SWC-135) 
- [CWE-20: Improper Input Validation](https://cwe.mitre.org/data/definitions/20.html) 
- [OpenZeppelin – Input Validation Utilities](https://docs.openzeppelin.com/contracts/4.x/api/utils#Address) 
- [Solidity Security Considerations – Input Validation](https://docs.soliditylang.org/en/latest/security-considerations.html#input-validation) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Missing Input Validation 
severity: H
score:
impact: 4    
exploitability: 3 
reachability: 4   
complexity: 2    
detectability: 5  
finalScore: 3.75
```

---

## 📄 Justifications & Analysis

- **Impact**: May lead to corrupted state or bypassed authorization
- **Exploitability**: Easily invoked with zero address or malformed input
- **Reachability**: Found in almost every protocol with user input
- **Complexity**: Simple mistake but dangerous in critical flows
- **Detectability**: Easily caught by static analyzers or audit review