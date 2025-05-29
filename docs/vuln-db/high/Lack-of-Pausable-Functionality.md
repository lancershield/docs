# Lack of Pausable Functionality

```YAML
id: TBA
title: Lack of Pausable Functionality
baseSeverity: H
category: emergency-control
language: solidity
blockchain: [ethereum]
impact: Protocol cannot be halted during critical failures
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.5.0", "<latest"]
cwe: CWE-693
swc: SWC-131
```
## 📝 Description

- Smart contracts deployed without a pausable mechanism (also known as a circuit breaker) lack the ability to halt critical functions during emergencies such as:
- Active exploits or zero-day vulnerabilities
- Oracle manipulation or price feed failures
- Governance takeovers
- Critical logic bugs or misconfigurations
- The absence of pause()/unpause() functionality exposes protocols to uncontrollable cascading losses or prolonged attack windows. Attackers can continue draining funds or exploiting logic even after the issue is discovered, due to lack of mitigation levers.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract LendingPool {
    mapping(address => uint256) public deposits;

    function deposit() external payable {
        deposits[msg.sender] += msg.value;
    }

    function withdraw(uint256 amount) external {
        require(deposits[msg.sender] >= amount, "Insufficient");
        deposits[msg.sender] -= amount;
        payable(msg.sender).transfer(amount); // ❌ No emergency switch
    }
}
```

## 🧪 Exploit Scenario

1. A reentrancy or price oracle bug is discovered in the protocol.
2. Before a fix can be deployed, attackers begin exploiting the vulnerability.
3. The dev team cannot pause withdraw() or deposit(), allowing continued abuse.
4. Millions in funds are drained despite detection.

**Assumptions:**

- The protocol operates in a fully open, non-upgradable context.
- No whenNotPaused modifier is used to gate sensitive functions.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeLendingPool is Pausable, Ownable {
    mapping(address => uint256) public deposits;

    function pause() external onlyOwner {
        _pause();
    }

    function unpause() external onlyOwner {
        _unpause();
    }

    function deposit() external payable whenNotPaused {
        deposits[msg.sender] += msg.value;
    }

    function withdraw(uint256 amount) external whenNotPaused {
        require(deposits[msg.sender] >= amount, "Insufficient");
        deposits[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Assets and core functions remain active during emergencies."
- context: "Public DeFi protocol with no time-lock or guardian"
  severity: C
  reasoning: "Unmitigated risk of mass exploitation if a bug is found."
- context: "Private contract with controlled access"
  severity: L
  reasoning: "Lower urgency due to centralized control and user trust."
```

## 🛡️ Prevention

### Primary Defenses

- Integrate Pausable from OpenZeppelin or custom circuit breaker.
- Apply whenNotPaused to all critical mutating functions (transfer, withdraw, etc.).

### Additional Safeguards

- Allow pause/unpause only through multisig or DAO-controlled role.
- Emit Paused()/Unpaused() events for off-chain monitoring.
- Add time locks to restrict rapid toggling.

### Detection Methods

- Scan for state-mutating functions without whenNotPaused.
- Verify absence of pause()/unpause() or _paused state variables.
- Tools: Slither (missing-pausable), manual audit, OpenZeppelin Defender

## 🕰️ Historical Exploits

- **Name:** The DAO Exploit 
- **Date:** 2016 
- **Loss:** Approximately 3.6 million ETH (~$60 million at the time) 
- **Post-mortem:** [Link to post-mortem](https://hackernoon.com/understanding-the-dao-hack-j8d3z) 
  
## 📚 Further Reading

- [SWC-136: Unobservable Functionality](https://swcregistry.io/docs/SWC-136/) 
- [OpenZeppelin Pausable](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable)  
- [OpenZeppelin Defender](https://docs.openzeppelin.com/defender/) 

---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Lack of Pausable Functionality 
severity: H 
score:
impact: 4  
exploitability: 2 
reachability: 4 
complexity: 1  
detectability: 5  
finalScore: 3.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Without pause capability, protocols are defenseless in active exploits.
- **Exploitability**: Not directly exploitable alone, but amplifies real vulnerabilities.
- **Reachability**: Affects nearly all contracts missing a circuit breaker pattern.
- **Complexity**: Trivial fix using OpenZeppelin; lack of it is avoidable.
- **Detectability**: Readily caught by auditors and tools like Slither or Defender.