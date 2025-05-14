# Async Withdraw Race Condition

```YAML

id: TBA
title: Async Withdraw Race Condition via Double Spend
severity: H
category: race-condition
language: solidity
blockchain: [ethereum]
impact: Multiple withdrawals from the same balance before state update
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-362
swc: SWC-107

```

## 📝 Description

- Double Spend via Async Withdraw refers to a race condition vulnerability where a user (or attacker) is able to call a `withdraw()` function multiple times before their balance is updated, especially when external calls (e.g., `transfer()`, `call()`) are made before state updates. This results in multiple successful withdrawals from the same balance or claim allocation.
- This often occurs in poorly ordered code in `withdraw()` or `claim()` functions, particularly when gas forwarding or reentrancy opportunities exist.

## 🚨 Vulnerable Code

```solidity
contract AsyncVault {
    mapping(address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        // ❌ External call happens before balance update
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Transfer failed");

        balances[msg.sender] = 0; // ❌ State update after call
    }
}
```
## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker deposits 1 ETH into AsyncVault.
2. They call withdraw(), which sends funds via call() before setting balance to 0.
3. The fallback function in attacker’s contract re-enters withdraw(), repeating the process.
4. Attacker drains funds multiple times, exceeding their original balance.

**Assumptions:**

- Contract uses external calls before updating internal state.
- No reentrancy guard or state-locking mechanism is in place.

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureVault is ReentrancyGuard {
    mapping(address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external nonReentrant {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        balances[msg.sender] = 0; // ✅ State updated before external call
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Transfer failed");
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always follow the Checks-Effects-Interactions pattern.
- Set balances or state before performing external calls.
- Use ReentrancyGuard (mutex lock) for functions that move ETH or tokens.

### Additional Safeguards

- Emit withdrawal events after success for off-chain monitoring.
- Consider using pull payment models (withdraw() pattern) rather than send() or push payments.
- Harden fallback functions and avoid sending gas-forwarding call() unless necessary.

### Detection Methods

- Slither: reentrancy-eth, dos, and external-call-before-state-change detectors.
- Manual inspection of withdraw(), claim(), or transfer() logic.
- Fuzz testing with reentrant contracts to simulate race conditions.

## 🕰️ Historical Exploits

- **Name:** The DAO Hack 
- **Date:** 2016-06 
- **Loss:** ~$60M 
- **Post-mortem:** [Link to post-mortem](https://blog.slock.it/the-dao-hack-explained-62429dbabf62) 
  

## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107)
-  [Solidity Docs – Checks-Effects-Interactions Pattern](https://soliditylang.org/security-considerations.html#use-the-checks-effects-interactions-pattern) 
-  [OpenZeppelin – ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) 
  

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Async Withdraw Race Condition 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.2
```


---

## 📄 Justifications & Analysis

- **Impact**: Enables attackers to extract more than their rightful balance.
- **Exploitability**: Reentrancy through fallback or low-level call is common and easy to deploy.
- **Reachability**: Very common in financial contracts with external withdrawals.
- **Complexity**: Simple logic error—attacker can act with basic tooling.
- **Detectability**: Readily flagged by tools like Slither and through code review.
