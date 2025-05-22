# Flashloan-Reentrancy in DeFi Aggregators

```YAML
id: TBA
title: Flashloan-Reentrancy in DeFi Aggregators 
severity: C
category: reentrancy
language: solidity
blockchain: [ethereum]
impact: Profit manipulation, asset duplication, or draining of aggregator pools
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-841
swc: SWC-107
```

## 📝 Description

- Many DeFi aggregators (e.g., DEX routers, yield optimizers, lending routers) process token operations such as swap(), deposit(), or repay() through composable contract logic. 
- If external callbacks or token transfers during these operations are vulnerable to flashloan-triggered reentrancy, an attacker can:
- Reenter aggregator logic while the internal state is still updating
- Reprice liquidity or borrow terms mid-transaction
- Exploit assumptions made about balances, prices, or LP states
- This vulnerability is especially critical in protocols that:
- Trust msg.sender without locks
- Call out to user-specified tokens or vaults
- Support composable actions like flashloan + deposit + withdraw in one call

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IERC20 {
    function transferFrom(address, address, uint256) external returns (bool);
}

contract DeFiAggregator {
    mapping(address => uint256) public userDeposits;

    function deposit(address token, uint256 amount) external {
        // ❌ vulnerable to reentrancy from malicious token contract
        IERC20(token).transferFrom(msg.sender, address(this), amount);
        userDeposits[msg.sender] += amount;
    }

    function withdraw(address token, uint256 amount) external {
        require(userDeposits[msg.sender] >= amount, "Insufficient");
        userDeposits[msg.sender] -= amount;
        IERC20(token).transferFrom(address(this), msg.sender, amount);
    }
}
```

## 🧪 Exploit Scenario

1. Attacker contracts borrow via a flashloan protocol.
2. During the transferFrom() inside deposit(), the malicious token triggers a callback to attacker logic.
3. The attacker reenters withdraw() or deposit() before the first call updates their balance.
4. Aggregator logic assumes the balance is unchanged and double-counts assets or miscalculates vault shares.
5. Flashloan is repaid with profit, and aggregator suffers loss or imbalance.

**Assumptions:**

- Aggregator logic calls into external contracts (ERC20, vaults) mid-operation.
- There is no nonReentrant or state lock.
- No check-effects-interactions pattern applied.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SafeAggregator is ReentrancyGuard {
    mapping(address => uint256) public userDeposits;

    function deposit(address token, uint256 amount) external nonReentrant {
        userDeposits[msg.sender] += amount; // ✅ update state first
        require(IERC20(token).transferFrom(msg.sender, address(this), amount), "Transfer failed");
    }

    function withdraw(address token, uint256 amount) external nonReentrant {
        require(userDeposits[msg.sender] >= amount, "Insufficient");
        userDeposits[msg.sender] -= amount;
        require(IERC20(token).transfer(msg.sender, amount), "Transfer failed");
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Protect external-facing functions using nonReentrant.
- Update all internal state before calling out to token contracts.

### Additional Safeguards

- Validate token contracts (especially those passed in by users) for standard ERC20 compliance.
- Use whitelists for allowed tokens where feasible.

### Detection Methods

- Look for external calls (ERC20, vaults, hooks) before state updates in logic
- Tools: Slither (reentrancy-vulnerability, external-calls-before-state-update), MythX, Foundry fuzzing

## 🕰️ Historical Exploits

- **Name:** Cream Finance Flashloan Loop 
- **Date:** 2021-10 
- **Loss:** ~$130M 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/cream-rekt/)
  
## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107/)
- [OpenZeppelin ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Flashloan-Reentrancy in DeFi Aggregators 
severity: C 
score:
impact: 5  
exploitability: 4 
reachability: 4   
complexity: 3  
detectability: 4  
finalScore: 4.35
```

---

## 📄 Justifications & Analysis

- **Impact**: Vaults or aggregators can be drained or manipulated due to improper trust of external contracts.
- **Exploitability**: Easily triggered with composable DeFi legos and malicious tokens.
- **Reachability**: Widespread pattern in DEX routers, yield farms, and flashloan-integrated strategies.
- **Complexity**: Requires attacker control over callback or token logic.
- **Detectability**: Tools can catch reentrancy paths but false negatives are possible without modeling full control flow.