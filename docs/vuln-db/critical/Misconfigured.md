# Misconfigured Role-Based Access Control

```YAML
id: TBA
title: Misconfigured Role-Based Access Control (RBAC)
baseSeverity: C
category: access-control
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Unauthorized privilege escalation or permanent loss of control
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-269
swc: SWC-105
```

## 📝 Description

- Misconfigured Role-Based Access Control (RBAC) occurs when roles defined within a smart contract using AccessControl (e.g., OpenZeppelin’s RBAC model) are incorrectly set, assigned, or checked, resulting in unauthorized users gaining access to sensitive functions. Common mistakes include:
- Assigning sensitive roles (like DEFAULT_ADMIN_ROLE) to unintended addresses.
- Forgetting to revoke roles after setup.
- Improper use of hasRole checks or granting roles within constructors without verification.
- This flaw can be exploited to mint tokens, modify state, drain treasuries, or escalate privileges.

## 🚨 Vulnerable Code

```solidity

contract Token is AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Project deploys the contract without revoking DEFAULT_ADMIN_ROLE or setting it to a secure multisig.
2. Deployer’s private key is leaked, or admin is forgotten/unused.
3. Attacker gains access to the deployer address and uses it to:
4. Grant themselves MINTER_ROLE.
5. Mint unlimited tokens.
6. Drain vaults or manipulate access to other protected functions.

**Assumptions:**

- Developer assumes the deployer won’t need admin control post-deployment.
- No role transfer or locking mechanism is in place.
- Admin role controls other roles (MINTER_ROLE, PAUSER_ROLE, etc.).

## ✅ Fixed Code

```solidity

contract Token is AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor(address multisigAdmin, address initialMinter) {
        _grantRole(DEFAULT_ADMIN_ROLE, multisigAdmin);
        _grantRole(MINTER_ROLE, initialMinter);
    }

    function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Production contract with exposed mint, upgrade, or withdraw logic"
  severity: C
  reasoning: "Complete compromise of contract's state or funds possible"
- context: "Testnet contract or function gated by non-sensitive role"
  severity: L
  reasoning: "Impact isolated or non-critical"
- context: "Role correctly assigned and access restricted to known multisig"
  severity: I
  reasoning: "Mitigated by governance controls"
```

## 🛡️ Prevention

### Primary Defenses

- Avoid assigning DEFAULT_ADMIN_ROLE to EOA deployers.
- Use multisigs or timelocked governance modules for role administration.
- Always lock down unused or temporary roles via _revokeRole.

### Additional Safeguards

- Use constructor parameters to explicitly assign roles.
- Periodically audit getRoleAdmin() outputs to ensure no privilege drift.
- Use access tiers for separation of duties (minting, pausing, governance, etc.).

### Detection Methods

- Slither: access-control, misconfigured-authorization, arbitrary-role-assignment detectors.
- Hardhat/Foundry tests for role abuse and boundary violations.
- Manual review of grantRole, revokeRole, hasRole logic across modules.

## 🕰️ Historical Exploits

- **Name:** SDAO Token Minter Role Bug 
- **Date:** 2022 
- **Loss:** Unlimited token minting possible (fixed before exploit) 
- **Post-mortem:** [Link to post-mortem](https://www.certik.com/projects/singularitydao) 
  
## 📚 Further Reading

- [SWC-105: Unprotected Function – SWC Registry](https://swcregistry.io/docs/SWC-105/) 
- [OpenZeppelin: AccessControl Docs](https://docs.openzeppelin.com/contracts/4.x/access-control) 
-  [Access Control in Solidity Smart Contracts - Metana](https://metana.io/blog/access-control-in-solidity-smart-contracts/) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Misconfigured Role-Based Access Control (RBAC) 
severity: C
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Privilege escalation and unauthorized minting or admin actions.
- **Exploitability**: Easy if attacker controls any default admin or role admin.
- **Reachability**: High — often exposed via public grantRole() calls.
- **Complexity**: Low — just one wrong role assignment is enough.
- **Detectability**: Detectable via audit, but common in quick deployments or forks.

