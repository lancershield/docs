# Emergency Function Abuse

```YAML
id: LS11H
title: Emergency Function Abuse 
baseSeverity: H
category: privileged-function
language: solidity
blockchain: [ethereum]
impact: Asset withdrawal, pausing, or privilege escalation without proper governance
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-284
swc: SWC-105
```

## 📝 Description

- Emergency function abuse occurs when privileged or high-impact functions intended for protocol recovery or admin override—such as `emergencyWithdraw()`, `pause()`, `rescueFunds()`—are:
- Exposed publicly or,
- Lack access control or,
- Are callable by misconfigured contracts (e.g., EOAs, not multisigs).
- These functions, if not properly secured, allow attackers or rogue admins to unilaterally:
- Drain assets,
- Bypass business logic (e.g., unstake without penalty),
- Freeze user funds,
- Pause/unpause the system arbitrarily.

## 🚨 Vulnerable Code

```solidity
contract EmergencyBackdoor {
    address public admin;

    constructor() {
        admin = msg.sender;
    }

    // ❌ No access control
    function emergencyWithdraw(address token, uint256 amount) external {
        IERC20(token).transfer(msg.sender, amount);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Contract includes an emergency escape hatch intended only for the admin.
2. Function emergencyWithdraw() lacks any modifier or admin check.
3. Any attacker can call it and withdraw arbitrary tokens from the contract.
4. Assets are permanently lost or protocol drained.

**Assumptions:**

- No onlyOwner, onlyAdmin, or require(msg.sender == admin) present.
- Emergency logic is reachable by any external caller.

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeEmergency is Ownable {
    function emergencyWithdraw(address token, uint256 amount) external onlyOwner {
        IERC20(token).transfer(msg.sender, amount);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Emergency function allows global withdrawal or fund movement"
  severity: H
  reasoning: "Leads to potential rugpull, loss of protocol funds"
- context: "Function scoped per-user but lacks rate limiting or review"
  severity: M
  reasoning: "Moderate risk of abuse in panic scenarios"
- context: "Function guarded by strong access roles and scoped logic"
  severity: L
  reasoning: "Pattern is secure if combined with RBAC and limits"
```

## 🛡️ Prevention

### Primary Defenses

- Restrict all emergency functions using onlyOwner, onlyAdmin, or access control roles (AccessControl).
- Use multisig wallets (e.g., Gnosis Safe) for critical function execution.
- Add timelocks or circuit breakers on functions like pause(), drain(), or migrate().

### Additional Safeguards

- Emit detailed logs on every emergency operation.
- Expose emergency actions to on-chain governance or community veto mechanisms.
- Implement emergencyWithdraw() such that it can only target specific rescue scenarios (e.g., stuck tokens, dust balances).

### Detection Methods

- Slither: unprotected-functions, missing-authorization, dangerous-function detectors.
- Manual inspection of functions named emergencyWithdraw, rescueFunds, pause, sweep, etc.
- Test authorization failure paths for emergency methods.

## 🕰️ Historical Exploits

- **Name:** Uranium Finance Emergency Withdraw Exploit 
- **Date:** 2021 
- **Loss:** ~$50M 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/uranium-rekt/) 

## 📚 Further Reading

- [SWC-105: Authorization Through tx.origin](https://swcregistry.io/docs/SWC-105) 
- [OpenZeppelin – AccessControl and Ownable](https://docs.openzeppelin.com/contracts/4.x/access-control) 
- [SCWE-014: Lack of Emergency Stop Mechanism – OWASP Smart Contract Security](https://scs.owasp.org/SCWE/SCSVS-CODE/SCWE-014/) 
- [Extorsionware: Exploiting Smart Contract Vulnerabilities for Fun and Profit](https://arxiv.org/pdf/2203.09843.pdf)

---

## ✅ Vulnerability Report

```markdown
id: LS11H
title: Emergency Function Abuse 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Grants control over critical functions—can lead to full asset theft or denial of service.
- **Exploitability**: Direct access with no protection = immediate compromise.
- **Reachability**: Emergency functions are often deployed but overlooked in audits.
- **Complexity**: Very simple attack; just a single call if authorization is missing.
- **Detectability**: Commonly flagged by Slither and manual inspection.