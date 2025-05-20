# Protected Variables Misused Due to Incorrect Access Control

```YAML
id: TBA
title: Protected Variables Misused Due to Incorrect Access Control
severity: H
category: access-control
language: solidity
blockchain: [ethereum]
impact: Unauthorized state manipulation or fund loss
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-284
swc: SWC-105
```

## 📝 Description

- In Solidity, declaring a variable as protected (e.g., internal or protected by a modifier) does not automatically enforce strict access control.
- If access control is implemented incorrectly or inconsistently—such as through weak custom modifiers, public setters, or improper inheritance—attackers may still read, overwrite, or manipulate the protected variable.
- This vulnerability commonly arises in contracts where:
- Ownership or roles are stored in internal variables but exposed through unsafe upgrade logic or proxy contracts.
- Protected values are overwritten by insecure setX() functions
  msg.sender is not properly checked in access control logic.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Vault {
    address internal owner;

    function setOwner(address _newOwner) external {
        // ❌ No access control
        owner = _newOwner;
    }

    function withdraw() external {
        require(msg.sender == owner, "Not authorized");
        // transfer logic...
    }
}
```

## 🧪 Exploit Scenario

1. A contract defines an internal variable owner.
2. A public/external setter function allows modifying it without restriction.
3. Attacker calls setOwner(attacker), gaining full control.
4. Attacker uses withdraw() to drain funds.

**Assumptions:**

- Contract uses variables that appear "protected" due to visibility or naming.
- Access control modifiers or validations are missing or bypassable.
- Functionality depends on trust in these variables (e.g., roles, permissions).

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SecureVault {
    address internal owner;

    modifier onlyOwner() {
        require(msg.sender == owner, "Not authorized");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function setOwner(address _newOwner) external onlyOwner {
        owner = _newOwner;
    }

    function withdraw() external onlyOwner {
        // transfer logic...
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always use modifiers like onlyOwner, onlyRole, or OpenZeppelin’s AccessControl.
- Avoid exposing internal variables through unprotected functions.

### Additional Safeguards

- Mark critical state as private unless access is explicitly needed.
- Use upgrade-safe initialization with validation.
- Validate proxy initialization to prevent storage slot overwrites.

### Detection Methods

- Check for unprotected public setters on internal/private state.
- Scan for access modifiers that fail to enforce msg.sender checks.
- Tools: Slither (unprotected-functions, incorrect-visibility), MythX, manual audit

## 🕰️ Historical Exploits

- **Name:** YAM Finance Governance Bug 
- **Date:** August 2020 
- **Loss:** ~$750K in locked funds 
- **Post-mortem:** [Link to post-mortem](https://medium.com/yam-finance/yam-minting-incident-postmortem-b76afd1aef45) 
- **Name:** Swerve Finance Admin Takeover Attempt 
- **Date:** March 2023 
- **Loss:** None (issue mitigated before exploitation) 
- **Post-mortem:** [Link to post-mortem](https://www.halborn.com/blog/post/explained-the-swerve-finance-hack-march-2023) 
  
## 📚 Further Reading

- [SWC-105: Unprotected Ether Withdrawal](https://swcregistry.io/docs/SWC-105/) 
- [OpenZeppelin Access Control Guide](https://docs.openzeppelin.com/contracts/4.x/access-control)
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Protected Variables Misused Due to Incorrect Access Control
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 3   
complexity: 2     
detectability: 3  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Direct state corruption, unauthorized withdrawals, or role escalation.
- **Exploitability**: Easily exploitable if setter is exposed and unguarded.
- **Reachability**: Anyone can call public functions unless restricted by modifier.
- **Complexity**: Relies on incorrect assumptions about visibility implying protection.
- **Detectability**: Often missed unless explicitly reviewing setter functions.







