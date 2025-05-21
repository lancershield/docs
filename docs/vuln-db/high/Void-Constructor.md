# Void Constructor Declared as Function Causes Missing Initialization Logic

```YAML
id: TBA
title: Void Constructor Declared as Function Causes Missing Initialization Logic
severity: H
category: constructor-misdefinition
language: solidity
blockchain: [ethereum]
impact: Critical variables uninitialized, ownership loss, or logic failure
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.0"]
cwe: CWE-665
swc: SWC-118
```

## 📝 Description

- In versions of Solidity prior to 0.8.0, constructors are defined using a function with the same name as the contract. 
- If a developer renames the contract and forgets to update the constructor name, the function becomes a public function—not a constructor. This results in:
- The intended constructor logic not being executed at deployment
- Public re-execution of sensitive logic like owner = msg.sender
- Uninitialized state or ownership hijack vulnerabilities
- This issue was one of the most severe errors in Solidity contract design prior to the introduction of the constructor keyword in Solidity ≥ 0.4.22 and standardized behavior in 0.8.x.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.4.24;

contract SecureVault {
    address public owner;

    // ❌ This is not a constructor; it's a public function
    function Vault() public {
        owner = msg.sender;
    }

    function withdraw() public {
        require(msg.sender == owner, "Not owner");
        // withdraw logic
    }
}
```

## 🧪 Exploit Scenario

1. Developer renames contract Vault to contract SecureVault but forgets to update the constructor function name.
2. The function Vault() becomes a public method.
3. The contract is deployed with owner = 0x0, or the constructor logic is skipped.
4. An attacker calls Vault() after deployment and becomes the new owner.
5. The attacker calls withdraw() and drains the contract.

**Assumptions:**

- Contract uses Solidity version < 0.8.0 with legacy constructor syntax.
- Constructor name and contract name mismatch goes unnoticed.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SecureVault {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function withdraw() public {
        require(msg.sender == owner, "Not owner");
        // withdraw logic
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always use the constructor keyword in Solidity ≥ 0.4.22.
- Avoid legacy constructor naming patterns.

### Additional Safeguards

- Set owner and other sensitive variables using modifiers or deployment scripts with verification.
- Restrict public functions that may overlap with legacy constructor names.

### Detection Methods

- Detect public functions that share common legacy constructor patterns.
- Compare function names to the contract name under compilers < 0.8.0.
- Tools: Slither (uninitialized-state), MythX, manual review

## 🕰️ Historical Exploits

- **Name:** Parity Wallet Library Bug 
- **Date:** 2017 
- **Loss:** $280M frozen 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert.html) 
- **Name:** Rubixi Constructor Vulnerability 
- **Date:** 2016 
- **Loss:** ~50 ETH stolen by exploiting public "constructor"
- **Post-mortem:** [Link to post-mortem](https://theethereum.wiki/w/index.php/Rubixi)
  
## 📚 Further Reading

- [SWC-118: Incorrect Constructor Name](https://swcregistry.io/docs/SWC-118/)
- [Solidity Docs – Constructors](https://docs.soliditylang.org/en/latest/contracts.html#constructors) 
- [Slither Detector – Unprotected Constructor](https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-constructor-name)
- [Rubixi Vulnerability Thread](https://ethereum.stackexchange.com/questions/1321/what-happened-to-rubixi-formerly-dynamic-pyramid) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Void Constructor Declared as Function Causes Missing Initialization Logic
severity: H
score:
impact: 5  
exploitability: 4 
reachability: 4  
complexity: 1    
detectability: 4  
finalScore: 4.0
```

## 📄 Justifications & Analysis

- **Impact**: Leads to complete ownership takeover or critical uninitialized variables.
- **Exploitability**: Public function allows any user to trigger constructor logic post-deploy.
- **Reachability**: Common in old contracts or forks using legacy patterns.
- **Complexity**: Very low; arises from forgotten renaming or outdated syntax.
- **Detectability**: Easily found with linters or static tools, especially under old compiler versions.