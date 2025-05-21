# Unimplemented Functions

```YAML
id: TBA
title: Unimplemented Functions
severity: M
category: interface-violation
language: solidity
blockchain: [ethereum]
impact: Broken integrations, failed calls, or misleading behavior
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-672
swc: SWC-123
```

## 📝 Description

- In Solidity, when a contract inherits from an interface or abstract contract, it is expected to implement all declared functions. 
- If a required function is declared but left unimplemented, the contract:
- Cannot be deployed (if marked abstract)
- Appears to conform to an interface but fails at runtime
- Breaks integrations expecting that function to work
- May mislead developers or off-chain indexers that rely on ABI definitions
- In some cases, the function is defined but left with an empty body, misleading users into thinking it performs meaningful logic when it does not.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IStrategy {
    function withdraw(uint256 amount) external;
}

contract BrokenStrategy is IStrategy {
    // ❌ Function declared but not implemented
    function withdraw(uint256 amount) external {
        // no logic – silently fails
    }
}
```

## 🧪 Exploit Scenario

1. A DeFi vault uses IStrategy.withdraw(amount) to request asset withdrawals.
2. The strategy contract inherits from IStrategy but implements withdraw() with no logic.
3. The vault assumes funds will be returned and updates balances accordingly.
4. No funds are transferred; vault and strategy go out of sync.
5. Users cannot withdraw, and rewards or rebalancing fails silently.

**Assumptions:**

- A required function is either completely missing or exists but lacks logic.
- No tests or runtime assertions validate function effect.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeStrategy is IStrategy {
    address public vault;
    IERC20 public token;

    constructor(address _vault, IERC20 _token) {
        vault = _vault;
        token = _token;
    }

    function withdraw(uint256 amount) external override {
        require(msg.sender == vault, "Not vault");
        require(token.transfer(vault, amount), "Transfer failed");
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use the abstract keyword if a contract is not meant to be fully implemented.
- Implement all functions of inherited interfaces with valid logic.
- Mark incomplete functions with revert("Unimplemented") if temporarily stubbed.

### Additional Safeguards

- Use tests to confirm interface behavior is correctly executed.
- Validate external integrations with testnet deployments and mocks.

### Detection Methods

- Check for interfaces with missing or empty method implementations.
- Audit inheritance trees and compiler warnings for abstract function mismatches.
- Tools: Slither (missing-inheritance, unimplemented-interface), solc warnings, Foundry/Hardhat coverage

## 🕰️ Historical Exploits

- **Name:** FakeERC20s in Aggregators 
- **Date:** 2021 
- **Loss:** ~$500K in failed trades via dummy contract with unimplemented transfer 
- **Post-mortem:** [Link to post-mortem](https://medium.com/1inch-network)
  
## 📚 Further Reading

- [SWC-123: Requirement Violation](https://swcregistry.io/docs/SWC-123/) 
- [Solidity Docs – Interfaces](https://docs.soliditylang.org/en/latest/contracts.html#interfaces) 
- [Slither – Missing Interface Implementation](https://github.com/crytic/slither/wiki/Detector-Documentation#functions-that-do-not-implement-an-interface) 

---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Unimplemented Functions
severity: M
score:
impact: 3       
exploitability: 2 
reachability: 4  
complexity: 1   
detectability: 5  
finalScore: 3.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Results in broken contract flows, invisible logic gaps, or user-facing errors.
- **Exploitability**: Not directly exploitable but allows attackers to take advantage of failed logic assumptions.
- **Reachability**: Common in strategies, upgradable modules, or factory contracts.
- **Complexity**: Basic oversight—easily fixable with standard Solidity practices.
- **Detectability**: Compiler will warn or error; easily found in manual or static reviews.