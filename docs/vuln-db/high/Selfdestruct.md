# Selfdestruct Vulnerabilities

```YAML
id: TBA
title: Selfdestruct Vulnerabilities in Destructible Contracts
severity: H
category: selfdestruct
language: solidity
blockchain: [ethereum]
impact: Permanent loss of contract code and logic, denial of service, asset redirection
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-284
swc: SWC-106
```

## 📝 Description

- Selfdestruct vulnerabilities arise when a contract includes a `selfdestruct` call (or `suicide` in older versions) without proper access control or safeguards. 
- This can allow an attacker (or unintentional caller) to **permanently remove the contract bytecode from the blockchain**, making the contract unusable and potentially redirecting Ether to an attacker-controlled address. 
- In upgradeable or delegatecall patterns, `selfdestruct` in implementation logic can be catastrophic.

## 🚨 Vulnerable Code

```solidity
contract Destructible {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function destroy() public {
        selfdestruct(payable(owner)); // ❌ No access control
    }
}
```
## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Contract is deployed with Ether stored or as part of a larger system.
2. Any user calls destroy() since it lacks onlyOwner or require.
3. The contract's code is erased from the blockchain.
4. All future interaction with the contract fails, causing protocol failure or DoS.

**Assumptions:**

- No access control on the selfdestruct() logic.
- selfdestruct is called directly or via delegatecall.

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeDestruct is Ownable {
    function destroy() external onlyOwner {
        selfdestruct(payable(owner()));
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Never expose selfdestruct or delegatecall-enabled destruct logic without strict access control (onlyOwner, onlyGovernance).
- Avoid selfdestruct entirely in production-grade contracts—use proxy upgradeability instead.

### Additional Safeguards

- Use modifiers like onlyProxyAdmin for destructible upgradeable logic.
- Audit all delegatecall chains for reachable selfdestruct paths.
- Log Destructed events to track use of this mechanism.

### Detection Methods

- Slither: selfdestruct and dangerous-call detectors.
- Manual inspection for selfdestruct, delegatecall, and upgrade flows.
- Symbolic analysis for indirect reachability of destruct logic.

## 🕰️ Historical Exploits

- **Name:** Parity Wallet Library Self-Destruct Bug 
- **Date:** 2017-11-06 
- **Loss:** ~$150M frozen 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert-2/) 


## 📚 Further Reading

- [SWC-106: Unprotected Selfdestruct](https://swcregistry.io/docs/SWC-106) 
- [Solidity Docs – selfdestruct](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#deactivate-and-self-destruct) 
- [OpenZeppelin – Upgradeable Proxy Pattern](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Selfdestruct Vulnerabilities 
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

- **Impact**: Full contract destruction, disabling functionality and breaking integrations.
- **Exploitability**: Highly likely if exposed without onlyOwner or require.
- **Reachability**: Often present in upgrade paths, legacy destruct functions, or test contracts.
- **Complexity**: Low effort to exploit—just one call needed.
- **Detectability**: Detectable by tools like Slither or manual audits with grep selfdestruct.
