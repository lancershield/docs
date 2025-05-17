# Hardcoded Non-Essential Parameters

```YAML
id: TBA
title: Hardcoded Non-Essential Parameters 
severity: L
category: maintainability
language: solidity
blockchain: [ethereum]
impact: Inflexible protocol configuration and upgrade overhead
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-611
swc: SWC-131
```

## 📝 Description

- Hardcoded non-essential parameters refers to values (such as fees, durations, limits, and addresses) that are written directly into the smart contract code instead of being stored in state variables or constructor-initialized. While not an immediate security threat, this practice:
- Reduces flexibility and upgradability
- Requires redeployment for any configuration change,
- Makes the contract more prone to technical debt and governance friction.
- These parameters often include:
- Fee percentages
- Cooldown durations
- Token or treasury addresses
- Reward multipliers
- Admin configurations

## 🚨 Vulnerable Code

```solidity
contract HardcodedConfig {
    function getFee() public pure returns (uint256) {
        return 500; // ❌ Hardcoded 5% (basis points)
    }

    function cooldownPeriod() public pure returns (uint256) {
        return 86400; // ❌ Hardcoded 1-day delay
    }
}
```

## 🧪 Exploit Scenario

Step-by-step impact:

1. Protocol deploys with a hardcoded getFee() value of 5%.
2. Market conditions change, and the team decides to reduce it to 3%.
3. Since the value is hardcoded, a new contract deployment is required.
4. This causes upgrade delays, user confusion, or inconsistent frontend behavior.

**Assumptions:**

- The value is not mission-critical (e.g., not part of immutable trust assumptions).
- Governance or owner wants to change the value post-deployment.

## ✅ Fixed Code

```solidity

contract Configurable {
    uint256 public fee; // in basis points
    address public owner;

    event FeeUpdated(uint256 newFee);

    constructor(uint256 initialFee) {
        fee = initialFee;
        owner = msg.sender;
    }

    function setFee(uint256 newFee) external {
        require(msg.sender == owner, "Unauthorized");
        require(newFee <= 1000, "Too high"); // max 10%
        fee = newFee;
        emit FeeUpdated(newFee);
    }

    function getFee() public view returns (uint256) {
        return fee;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use state variables for all configurable parameters.
- Initialize parameters via the constructor or an initializer.
- Restrict updates using onlyOwner, onlyGovernance, or AccessControl.

### Additional Safeguards

- Emit events on config changes to ensure observability.
- Document acceptable ranges and constraints for each configurable value.
- In upgradable contracts, prefer UUPS pattern with secure admin control.

### Detection Methods

- Slither: hardcoded-constant, fixed-parameter, immutable-config detectors.
- Manual audit to identify return <literal> in pure/view functions with constants.
- Use grep or AST tools to find magic numbers or hardcoded literals.

## 🕰️ Historical Exploits

- **Name:** Compound Interest Rate Hardcoding 
- **Date:** 2020  
- **Loss:** No direct financial loss
- **Post-mortem:** [Link to post-mortem](https://docs.compound.finance/interest-rates/)

## 📚 Further Reading

- [SWC-131: Presence of Unused or Hardcoded Code](https://swcregistry.io/docs/SWC-131) 
- [Solidity Docs – Constants vs Variables](https://docs.soliditylang.org/en/latest/contracts.html#constants) 
- [OpenZeppelin – Upgradeable Patterns](https://docs.openzeppelin.com/upgrades-plugins)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Hardcoded Non-Essential Parameters 
severity: L
score:
impact: 2         
exploitability: 0 
reachability: 5   
complexity: 1     
detectability: 5  
finalScore: 2.1

```

---

## 📄 Justifications & Analysis

- **Impact**: Limits operational flexibility and causes protocol downtime during config changes.
- **Exploitability**: Not an attack vector on its own.
- **Reachability**: Common in early-stage and boilerplate contracts.
- **Complexity**: Simple to fix — requires moving constants to variables.
- **Detectability**: High — Slither and manual reviews can easily identify hardcoded logic.