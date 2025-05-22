# Privileged Role Migration Loophole

```YAML
id: TBA
title: Privileged Role Migration Loophole 
severity: C
category: access-control
language: solidity
blockchain: [ethereum]
impact: Loss of contract control, fund theft, or governance takeover
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-862
swc: SWC-105
```

## 📝 Description

- Many smart contracts include privileged roles such as owner, admin, or governor. In upgradeable or modular systems, migration from one privileged role to another must be done carefully to avoid:
- Loopholes where roles are transferred but old roles are still active
- Race conditions where a new admin gains control before revoking the previous one
- Missing role cleanup, leaving both roles with concurrent privileges
- Insecure initialization, where role-setting logic is left unprotected
- This vulnerability typically emerges in custom role systems or upgradeable contracts (e.g., proxy + logic) when transferOwnership() or setAdmin() is called without clearing prior authorities or without locking down initialization.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract RoleManager {
    address public owner;
    address public admin;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    modifier onlyAdmin() {
        require(msg.sender == admin, "Not admin");
        _;
    }

    function transferOwnership(address newOwner) external onlyOwner {
        owner = newOwner;
    }

    function setAdmin(address newAdmin) external onlyOwner {
        admin = newAdmin;
    }

    function emergencyWithdraw() external onlyAdmin {
        // withdraw logic
    }
}
```

## 🧪 Exploit Scenario

1. Project owner calls transferOwnership(newOwner) and assumes all control is migrated.
2. Old admin (malicious or disgruntled contributor) is still active.
3. They call emergencyWithdraw() and drain funds or interrupt protocol logic.
4. Alternatively, the new owner sets a new admin, but the old one was never removed or checked.
5. Multiple privileged roles now exist concurrently without a clear single source of truth.

**Assumptions:**

- Role transfer does not revoke old privileges.
- Admin permissions are not audited or centrally managed.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureRoleManager is Ownable {
    address public admin;

    modifier onlyAdmin() {
        require(msg.sender == admin, "Not admin");
        _;
    }

    function setAdmin(address newAdmin) external onlyOwner {
        require(newAdmin != address(0), "Zero address");
        admin = newAdmin;
    }

    function revokeAdmin() external onlyOwner {
        admin = address(0); // ✅ Explicit revocation of previous admin
    }

    function emergencyWithdraw() external onlyAdmin {
        require(admin != address(0), "Admin revoked");
        // withdraw logic
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use OpenZeppelin’s AccessControl or Ownable to manage roles formally.
- Always revoke old roles during migration (revokeRole, setX(address(0))).

### Additional Safeguards

- Prevent use of critical functions if any privileged role is undefined or stale.
- Audit all role transitions in tests and simulate re-entrance paths.

### Detection Methods

- Search for multiple privileged roles and their associated mutators.
- Check whether old role revocation is enforced during setRole() logic.
- Tools: Slither (reentrancy-eth, uninitialized-state), manual review of access control

## 🕰️ Historical Exploits

- **Name:** Lendf.Me DeFi Access Mismanagement 
- **Date:** 2020-04 
- **Loss:** ~$25M withdrawn due to improper admin and proxy role handling 
- **Post-mortem:** [Link to post-mortem](https://dforce.network/blog/post-mortem-analysis-of-lendfme-incident) 
  
## 📚 Further Reading

- [SWC-105: Unprotected Function](https://swcregistry.io/docs/SWC-105/) 
- [Solidity Docs – Ownable](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable) 
- [OpenZeppelin AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl)
- [Best Practices – Role-Based Access](https://consensys.github.io/smart-contract-best-practices/recommendations/#restrict-access-to-sensitive-functions)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Privileged Role Migration Loophole 
severity: C
score:
impact: 5    
exploitability: 4 
reachability: 4   
complexity: 3  
detectability: 3  
finalScore: 4.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Attackers or stale roles can hijack contract control or drain assets.
- **Exploitability**: Occurs if developers assume role transitions are atomic or exclusive.
- **Reachability**: Found in DAO, admin, or proxy systems with layered access.
- **Complexity**: Requires understanding multi-role lifecycle and invariants.
- **Detectability**: Requires human review or formal modeling of role transitions.