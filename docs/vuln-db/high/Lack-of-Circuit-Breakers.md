# Lack of Circuit Breakers

```YAML
id: LS55H
title: Lack of Circuit Breakers
baseSeverity: H
category: emergency-control
language: solidity
blockchain: [ethereum]
impact: Uncontrollable protocol drain or failure under duress
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.5.0", "<latest"]
cwe: CWE-693
swc: SWC-131
```

## 📝 Description

- Lack of circuit breakers for secondary features refers to the absence of conditional `pause`, `disable`, or `emergencyStop` mechanisms in non-core contract functions such as:
- Claim, redeem, migrate, swap, vote, bid, etc.
- While these features are not directly responsible for fund custody, their failure or abuse can:
- Lock user interactions
- Enable spam, bot abuse, or gas griefing
- Prevent recovery during upgrades, and
- Amplify damage when paired with other vulnerabilities.
- Without selective pause control, contracts lack the ability to quickly respond to partial threats.

## 🚨 Vulnerable Code

```solidity
contract FeatureHeavy {
    function redeem() external {
        // Logic to calculate and send tokens
    }

    function vote(uint256 proposalId) external {
        // Record user vote
    }
    // ❌ No way to disable redeem or vote during bugs/emergencies
}
```

## 🧪 Exploit Scenario

Step-by-step impact:

1. A logic bug is discovered in redeem() allowing users to withdraw excess rewards.
2. There is no circuit breaker or pause() mechanism tied to this function.
3. Attackers continue to exploit the feature while core functions remain intact.
4. Protocol cannot patch without full upgrade or redeploy.

**Assumptions:**

- Contract has auxiliary features with state impact.
- No fine-grained control to disable only specific functions.

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeFeatureControl is Pausable, Ownable {
    bool public redeemPaused;
    bool public votePaused;

    function pauseRedeem() external onlyOwner {
        redeemPaused = true;
    }

    function redeem() external {
        require(!redeemPaused, "Redeem paused");
        // Safe redeem logic
    }

    function vote(uint256 proposalId) external {
        require(!votePaused, "Voting paused");
        // Voting logic
    }

    function pauseVote() external onlyOwner {
        votePaused = true;
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Without pause, any bug may drain the protocol irreversibly."
- context: "Governance-controlled DeFi protocol"
  severity: C
  reasoning: "Failure to halt critical operations can lead to systemic loss."
- context: "Private contract with limited usage"
  severity: L
  reasoning: "Lower risk due to trusted actors and limited exposure."
```

## 🛡️ Prevention

### Primary Defenses

- Add feature-specific circuit breakers (e.g., redeemPaused, swapPaused) instead of global pause() only.
- Use require(!XPaused) in each critical function logic.
- Combine with OpenZeppelin’s Pausable or modular access control.

### Additional Safeguards

- Emit FeaturePaused(string feature) events for observability.
- Define protocol-wide governance procedures to toggle critical and secondary functions separately.
- Create monitoring bots to alert based on misuse rates of auxiliary functions.

### Detection Methods

- Slither: missing-feature-pause, no-emergency-guard, feature-overexposed detectors.
- Manual audit of all externally callable functions to verify whenNotPaused or equivalents exist.
- Coverage reports for require(!paused) or emergency triggers.

## 🕰️ Historical Exploits

- **Name:** The DAO Hack 
- **Date:** 2016-06-17 
- **Loss:** Approximately $60 million 
- **Post-mortem:** [Link to post-mortem](https://neptunemutual.com/blog/the-story-behind-the-dao-hack) 


## 📚 Further Reading

- [SWC-136: Unexpected Behavior / Missing Checks](https://swcregistry.io/docs/SWC-136) 
- [Solidity Docs – Pausable Pattern](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable) 
- [OpenZeppelin – Circuit Breakers](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks) 

---

## ✅ Vulnerability Report

```markdown
id: LS55H
title: Lack of Circuit Breakers  
severity: H
score:
impact: 4         
exploitability: 2 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 3.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Affects protocol recovery and integrity during active threats.
- **Exploitability**: Needs bug in feature logic but no defense mechanism.
- **Reachability**: Very common in protocols with redeem, vote, claim, or migrate.
- **Complexity**: Low — fix involves simple boolean flags and require guards.
- **Detectability**: High — caught easily with Slither or checklist-based audits.