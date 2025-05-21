# Missing Events in Access Control

```YAML
id: TBA
title: Missing Events in Access Control Functions Prevent Auditability and Off-Chain Traceability
severity: M
category: access-control
language: solidity
blockchain: [ethereum]
impact: Inability to detect permission changes or admin abuse
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-778
swc: SWC-136
```

## 📝 Description

- Access control functions (e.g., grantRole, revokeRole, transferOwnership, setAdmin, etc.) are critical to the integrity and governance of smart contracts. 
- If these functions do not emit events, it becomes difficult or impossible for:
- Users to verify role or permission changes
- Off-chain monitors and indexers to track admin activity
- Auditors to trace privileged function usage
- This does not affect on-chain logic directly, but undermines transparency and opens up avenues for stealthy governance abuse, rug pulls, or denial of service through undeclared permission revocations.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract AdminControl {
    address public admin;

    function setAdmin(address newAdmin) external {
        require(msg.sender == admin, "Not admin");
        admin = newAdmin; // ❌ No event emitted
    }
}
```

## 🧪 Exploit Scenario

1. A malicious admin deploys a governance contract without events in access control methods.
2. They silently change the admin address to a private wallet.
3. Off-chain monitoring tools fail to detect the change in real time.
4. The attacker executes privileged functions from the new address to drain funds or alter protocol parameters without visibility or audit trail.

**Assumptions:**

- Access control functions exist without emit logs.
- Protocol relies on external monitoring for trust minimization or security alerts.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeAdminControl {
    address public admin;

    event AdminChanged(address indexed oldAdmin, address indexed newAdmin);

    constructor() {
        admin = msg.sender;
    }

    function setAdmin(address newAdmin) external {
        require(msg.sender == admin, "Not admin");
        emit AdminChanged(admin, newAdmin); // ✅ Log the change
        admin = newAdmin;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always emit event logs for sensitive role/permission changes.
- Follow the OpenZeppelin pattern (OwnershipTransferred, RoleGranted, etc.).

### Additional Safeguards

- Include modifiers like onlyOwner or onlyRole to formalize permission structure.
- Make admin transitions two-step (proposeAdmin, acceptAdmin) with events in both steps.

### Detection Methods

- Search for admin = or role[...] = patterns without adjacent emit statements.
- Review permissioned function modifiers or admin setters for missing logs.
- Tools: Slither (missing-events), Solhint (event-name-casing), manual access control audit

## 🕰️ Historical Exploits
 
- **Name:** YAM Governance Invisibility Bug 
- **Date:** 2020 
- **Loss:** $750K+ at risk (eventless governance actions) 
- **Post-mortem:** [Link to post-mortem](https://medium.com/yam-finance) 
  
## 📚 Further Reading

- [SWC-136: Unobservable Behavior via Missing Events](https://swcregistry.io/docs/SWC-136/) 
- [Smart Contract Vulnerabilities 2 – Medium](https://medium.com/@bartubozkurt35/smart-contract-vulnerabilities-2-de08d0ac73c2) 
- [Solidity Documentation: Events and Logging](https://docs.soliditylang.org/en/latest/contracts.html#events) 
- [SolidityScan: Access Control Vulnerabilities in Smart Contracts](https://blog.solidityscan.com/access-control-vulnerabilities-in-smart-contracts-a31757f5d707)
- [Access Control Vulnerability and Solution in A Solidity Smart Contract – CoinsBench](https://coinsbench.com/access-control-vulnerability-and-solution-in-a-solidity-smart-contract-e4e06b0df2ee)
  
---
  
## ✅ Vulnerability Report

```markdown
id: TBA
title: Missing Events in Access Control Functions Prevent Auditability and Off-Chain Traceability
severity: M
score:
impact: 3 
exploitability: 2 
reachability: 4 
complexity: 1     
detectability: 5 
finalScore: 2.85
```

---

## 📄 Justifications & Analysis

- **Impact**: Omits critical security signal from off-chain observability tools.
- **Exploitability**: Allows stealth admin actions if coupled with other vulnerabilities.
- **Reachability**: Ubiquitous in custom governance and protocol control code.
- **Complexity**: Trivial developer mistake—lack of emit.
- **Detectability**: Easy to flag with static analysis tools or visual inspection.