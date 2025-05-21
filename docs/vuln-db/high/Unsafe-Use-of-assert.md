# Unsafe Use of assert

```YAML
id: TBA
title: Unsafe Use of assert 
severity: H
category: logic
language: solidity
blockchain: [ethereum]
impact: Contract failure, locked funds, or unusable state
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.10", "<=0.8.25"]
cwe: CWE-670
swc: SWC-123
```

## 📝 Description

- The assert() keyword in Solidity is meant to detect internal logic errors and impossible conditions. If an assert() fails:
- A Panic error (0x01) is thrown (since Solidity 0.8.x)
- All gas is consumed (not just remaining gas)
- The contract state is reverted
- More importantly, excessive or repeated assert() failures can cause smart contracts to become unusable, especially if assert() guards runtime-sensitive conditions (e.g., balance checks, external inputs).
- Using assert() to enforce input validation, permissions, or business rules is dangerous—it should be reserved for invariants only.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Vault {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) external {
        assert(balances[msg.sender] >= amount); // ❌ misuse of assert for user input
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }

    receive() external payable {
        balances[msg.sender] += msg.value;
    }
}
```

## 🧪 Exploit Scenario

1. A user accidentally (or maliciously) calls withdraw(1000) with a balance of only 500.
2. The assert() fails, consuming all gas and reverting with a Panic error.
3. In proxy systems or gas-sensitive environments, this behavior breaks fallback logic or drains gas.
4. If the contract includes many such assert() checks, it becomes unstable under edge-case conditions.

**Assumptions:**

- assert() is used where require() should be.
- External inputs can trigger the failing condition naturally.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeVault {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance"); // ✅ safe input check
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }

    receive() external payable {
        balances[msg.sender] += msg.value;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use require() for external-facing checks and user input validation.
- Use assert() only for checking internal invariants that should never fail unless there is a bug.

### Additional Safeguards

- Review all assert() statements in code and downgrade to require() unless checking an invariant.
- Add try/catch around external calls to prevent cascading assert() panics.

### Detection Methods

- Search for assert() applied to user-controlled values.
- Tools: Slither (incorrect-assert), MythX, custom static analyzers

## 🕰️ Historical Exploits

- **Name:** GovernMental DApp 
- **Date:** 2016 
- **Loss:** ETH locked due to failed `assert` and logic bugs
- **Post-mortem:** [Link to post-mortem](https://www.reddit.com/r/ethereum/comments/4np972/governmental_dapp_scam_or_honeypot/) 

---

## 📚 Further Reading

- [SWC-123: Requirement Violation](https://swcregistry.io/docs/SWC-123/) 
- [Solidity Docs – assert vs require](https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions)
- [Slither – Dangerous Usage of Assert](https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-use-of-assert)

--- 

## 🚨 Vulnerable Code

```markdown
id: TBA
title: Unsafe Use of assert 
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

- **Impact**: Panic errors consume all gas, degrade user experience, and may cause contract failures.
- **Exploitability**: Users can easily trigger failures if assert() checks inputs or balances.
- **Reachability**: Common in contracts where assert() is misunderstood or misused.
- **Complexity**: Simple misunderstanding of control flow semantics.
- **Detectability**: Highly visible to tools like Slither or visual inspection.
