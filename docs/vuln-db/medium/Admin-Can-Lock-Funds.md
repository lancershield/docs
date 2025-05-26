# Admin Can Lock Funds

```YAML
id: TBA
title: Admin Can Lock Funds 
baseSeverity: M
category: access-control
language: solidity
blockchain: [ethereum, bsc, arbitrum, optimism, polygon]
impact: Funds become inaccessible to users or permanently frozen
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-732
swc: SWC-124
```

## 📝 Description

- Certain contracts delegate critical permissions (e.g., pausing withdrawals, freezing assets, or setting withdrawal policies) to an admin or owner role. If these controls are:
- Unrestricted
- Permanently enabled
- Lack clear exit paths for users
- The admin can intentionally or unintentionally lock user funds without recourse. This centralization risk violates decentralization and liveness assumptions of DeFi.
- This vulnerability is critical when:
- Users stake, lock, or deposit funds expecting later withdrawal
- No unpause or timelock mechanisms exist
- Admin retains exclusive control over pause(), withdraw(), or setRecipient()

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract LockedVault {
    address public owner;
    bool public paused;
    mapping(address => uint256) public balances;

    constructor() {
        owner = msg.sender;
    }

    function pauseWithdrawals() external {
        require(msg.sender == owner, "Not owner");
        paused = true; // ❌ can pause indefinitely
    }

    function withdraw() external {
        require(!paused, "Withdrawals paused");
        uint256 bal = balances[msg.sender];
        balances[msg.sender] = 0;
        payable(msg.sender).transfer(bal);
    }

    receive() external payable {
        balances[msg.sender] += msg.value;
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Thousands of users deposit ETH into LockedVault.
2. The admin later calls pauseWithdrawals().
3. Since there’s no unpause() or escape hatch, users can’t recover funds.
4. Even if unintentional, the vault becomes a black hole without governance override.

**Assumptions:**

- Centralized admin control
- No DAO/governance for upgrade or unlock
- Users have no self-service recovery options

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeVault is Ownable, Pausable {
    mapping(address => uint256) public balances;

    function pauseWithdrawals() external onlyOwner {
        _pause();
    }

    function resumeWithdrawals() external onlyOwner {
        _unpause();
    }

    function withdraw() external whenNotPaused {
        uint256 bal = balances[msg.sender];
        require(bal > 0, "No funds");
        balances[msg.sender] = 0;
        payable(msg.sender).transfer(bal);
    }

    receive() external payable {
        balances[msg.sender] += msg.value;
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Admin pause with no unpause or fallback path"
  severity: M
  reasoning: "Funds can be locked indefinitely due to central control."
- context: "Admin pause with public unpause or time-bound recovery"
  severity: L
  reasoning: "Low severity if safety mechanisms exist and are transparent."
- context: "Pause uses DAO-based multisig or timelock for execution"
  severity: I
  reasoning: "Governance-based pause is unlikely to be abused."
```

## 🛡️ Prevention

### Primary Defenses

- Require pause() to be time-limited or governed by multisig/timelock
- Expose unpause() to DAO, multisig, or automated safety checks

### Additional Safeguards

- Allow users to force exit after grace period
- Log pausing events with reasons for transparency
- Add circuit-breaker logic that unlocks if admin inactivity is detected

### Detection Methods

- Identify pause/lock/disable functions without user exit fallback
- Tools: Slither (unrestricted-write, centralized-control), manual review

## 🕰️ Historical Exploits
 
- **Name:** Parity Multisig Wallet Freeze 
- **Date:** 2017-11 
- **Loss:** ~$300 
- **Post-mortem:** [Link to post-mortem](https://metaschool.so/articles/access-control-vulnerabilities-in-smart-contracts) 
  
## 📚 Further Reading

- [SWC-124: Arbitrary Location Write](https://swcregistry.io/docs/SWC-124/) 
- [CWE-732: Incorrect Permission Assignment](https://cwe.mitre.org/data/definitions/732.html) 
- [OpenZeppelin – Pausable](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable) 
   
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Admin Can Lock Funds 
severity: M
score:
impact: 5       
exploitability: 3 
reachability: 4   
complexity: 2   
detectability: 5 
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: All funds may be frozen with no redemption path
- **Exploitability**: Centralized admin has unilateral control
- **Reachability**: Common in treasury contracts and launchpad vaults
- **Complexity**: Mistake is simple and often overlooked
- **Detectability**: Obvious to reviewers, flagged by tooling
