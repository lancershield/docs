# Codex Pattern Vulnerability Enables Unauthorized Data Access

```YAML
id: TBA
title: Codex Pattern Vulnerability Enables Unauthorized Data Access or Mutation
severity: H
category: access-control
language: solidity
blockchain: [ethereum]
impact: Unauthorized reads/writes of shared state
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-284
swc: SWC-135
```

## 📝 Description

- The Codex pattern is a storage abstraction design commonly used in modular Solidity architectures where one or more contracts interact with a shared "Codex" storage contract (e.g., Codex.sol). While modular and gas-efficient, improper design or access control within the Codex contract can result in:
- Unrestricted access to critical mappings or structs
- Unauthorized updates to protocol state (e.g., vaults, balances, positions)
- Logic separation assumptions being broken
- This often happens when the Codex exposes public state mutation functions (e.g., modifyBalance, setDebt) without checking msg.sender, or fails to restrict writes to a known list of trusted modules.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Codex {
    mapping(address => uint256) public balances;

    function modifyBalance(address user, int256 delta) external {
        balances[user] = uint256(int256(balances[user]) + delta); // ❌ No access control
    }
}
```

## 🧪 Exploit Scenario

1. The protocol uses Codex to store collateral and debt balances.
2. A malicious actor calls modifyBalance(attacker, 1000000) directly.
3. The balance is updated without permission or validation.
4. The attacker now withdraws excess collateral or borrows funds illegitimately.

**Assumptions:**

- Protocol assumes Codex is only used by trusted modules (e.g., Vault, Borrower).
- There is no whitelist or role-checking on Codex modification functions.
- Attack surface is exposed through direct external access.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract Codex {
    mapping(address => uint256) public balances;
    mapping(address => bool) public trustedModules;
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyModule() {
        require(trustedModules[msg.sender], "unauthorized");
        _;
    }

    function addModule(address module) external {
        require(msg.sender == owner, "not owner");
        trustedModules[module] = true;
    }

    function modifyBalance(address user, int256 delta) external onlyModule {
        balances[user] = uint256(int256(balances[user]) + delta);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Implement strict access control for all Codex mutating functions.
- Treat Codex as internal storage layer, not a generic external contract.

### Additional Safeguards

- Use internal calls (delegatecall, call) gated by trusted modules.
- Separate public getters from mutation logic.
- Implement upgradeability carefully to prevent logic substitution attacks.

### Detection Methods

- Scan for external/public functions that mutate Codex state without require(msg.sender == X) or onlyRole(...).
- Check Codex contracts for missing trust boundaries.
- Tools: Slither (access-control, missing-auth), manual module inspection

## 🕰️ Historical Exploits

- **Name:** Visor Finance Exploit 
- **Date:** 2022 
- **Loss:** Unauthorized fund withdrawals due to improper interface implementation 
- **Post-mortem:** [Link to post-mortem](https://www.nethermind.io/blog/smart-contract-vulnerabilities-and-mitigation-strategies)
- **Name:** LAND Token Exploit 
- **Date:** 2024  
- **Loss:** Unauthorized privilege escalation via missing access control checks  
- **Post-mortem:** [Link to post-mortem](https://solidityscan.com/discover/owasp-smart-contract-top-10-security-risks-and-vulnerabilities-a-deep-dive-with-real-world-exploits-and-credshields-contribution/) 

## 📚 Further Reading

- [SWC-135: Incorrect Authorization](https://swcregistry.io/docs/SWC-135/) 
- [OpenZeppelin AccessControl Docs](https://docs.openzeppelin.com/contracts/4.x/access-control) 
- [Smart Contract Access Control Best Practices – Krayon Digital](https://www.krayondigital.com/blog/smart-contract-access-control-best-practices) 
- [OWASP Reveals Top 10 Smart Contract Vulnerabilities For 2025 – LinkedIn](https://www.linkedin.com/pulse/owasp-reveals-top-10-smart-contract-vulnerabilities-eyfte) 
  
---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Codex Pattern Vulnerability Enables Unauthorized Data Access or Mutation
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 3.45
```

---

## 📄 Justifications & Analysis

- **Impact**: Unchecked writes to Codex state break accounting and trust assumptions.
- **Exploitability**: Easy to exploit if functions are callable and unguarded.
- **Reachability**: Codex is public and callable in most modular systems.
- **Complexity**: Vulnerability is subtle due to logic separation across modules.
- **Detectability**: Easy to miss unless specifically validating access to Codex state mutators.

