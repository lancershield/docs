# Conflicting Modifiers

```YAML
id: TBA
title: Conflicting Modifiers 
baseSeverity: M
category: access-control
language: solidity
blockchain: [ethereum]
impact: Access control bypass or logic inconsistency
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-284
swc: SWC-135
```

## 📝 Description

- In Solidity, modifiers are used to prepend access control, validations, and logic constraints to functions. 
- When multiple modifiers are applied to a function, their order of execution matters. If the logic within these modifiers:
- Conflicts (e.g., both change state or assume exclusive conditions)
- Relies on internal state that is modified by another modifier
- Overrides error messages or conditional flow
- Then execution becomes ambiguous or insecure, leading to:
- Incorrect access granted
- Security conditions being skipped
- Broken assumptions in nested modifiers
- This vulnerability is especially dangerous when reusing OpenZeppelin's onlyOwner, whenNotPaused, or custom modifiers together without understanding how they interact.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract ConflictingModifiers {
    bool public paused = false;
    address public admin;

    modifier onlyAdmin() {
        require(msg.sender == admin, "Not admin");
        _;
    }

    modifier whenNotPaused() {
        require(!paused, "Paused");
        _;
    }

    constructor() {
        admin = msg.sender;
    }

    // ❌ Ambiguous modifier logic
    function emergencyWithdraw() external onlyAdmin whenNotPaused {
        // withdraw funds
    }

    function pause() external onlyAdmin {
        paused = true;
    }
}
```

## 🧪 Exploit Scenario

1. Admin pauses the contract using pause().
2. A vulnerability is found, and funds need to be moved using emergencyWithdraw().
3. Since whenNotPaused runs after onlyAdmin, the call fails—even for admin.
4. Emergency recovery fails, funds become trapped, or attackers exploit paused state.
5. Alternatively, if modifier order is reversed, and whenNotPaused is placed before a state-changing modifier, internal assumptions break (e.g., state updated during checks).

**Assumptions:**

- Multiple modifiers are applied together.
- Developers assume modifiers are context-independent.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeModifiers is Ownable, Pausable {
    constructor() {
        _pause(); // optional
    }

    function emergencyWithdraw() external onlyOwner {
        // ✅ No whenNotPaused — can bypass pause logic if needed
        // rescue funds logic
    }

    function normalWithdraw() external whenNotPaused {
        // standard user withdrawal logic
    }

    function pauseContract() external onlyOwner {
        _pause();
    }

    function unpauseContract() external onlyOwner {
        _unpause();
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Modifiers silently cancel each other causing logic bypass or function lock"
  severity: M
  reasoning: "Can freeze core functions or silently reduce authorization boundaries"
- context: "Modifiers just overlap logically but do not block access"
  severity: L
  reasoning: "Redundant but not harmful unless code changes"
- context: "Well-documented modifier interactions with explicit tests"
  severity: I
  reasoning: "Safe if intentional and verified"
```

## 🛡️ Prevention

### Primary Defenses

- Avoid combining modifiers that assume contradictory state conditions.
- Ensure modifier order of application is explicitly understood and documented.

### Additional Safeguards

- Write unit tests for all composite modifier paths.
- Create modifier variants for edge cases (onlyOwnerWhenPaused, onlyRecoveryAdmin, etc.).

### Detection Methods

- Review all externally callable functions with multiple modifiers.
- Analyze modifier logic for shared state mutation or dependency.
- Tools: Slither (modifier-order, modifier-conflict), MythX

## 🕰️ Historical Exploits

- **Name:** DIA Data NFT Audit 
- **Date:** 2021 
- **Loss:** ~$100,000 
- **Post-mortem:** [Link to post-mortem](https://content.diadata.org/wp-content/uploads/2021/09/02_Smart-Contract-Audit_DIA_DRMNFT.pdf)
- **Name:** Plume Protocol Audit 
- **Date:** 2024 
- **Loss:** ~$30,000 
- **Post-mortem:** [Link to post-mortem](https://www.halborn.com/audits/plume/plume-contracts)

## 📚 Further Reading

- [SWC-135: Incorrect Modifier Logic](https://swcregistry.io/docs/SWC-135/) 
- [Solidity Docs – Modifier Precedence](https://docs.soliditylang.org/en/latest/contracts.html#modifiers) 
- [OpenZeppelin Access Control](https://docs.openzeppelin.com/contracts/4.x/api/access) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Conflicting Modifiers 
severity: M
score:
impact: 4      
exploitability: 2 
reachability: 4  
complexity: 2    
detectability: 4  
finalScore: 3.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Logic may be skipped or improperly blocked due to modifier order.
- **Exploitability**: Exploitable if attacker relies on skipped checks or blocked emergency flows.
- **Reachability**: High—common in contracts with multiple inherited access patterns.
- **Complexity**: Medium—requires deep understanding of logic layering.
- **Detectability**: Detectable via Slither or detailed function review.