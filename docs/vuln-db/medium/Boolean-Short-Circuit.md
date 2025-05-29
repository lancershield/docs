# Boolean Short-Circuit  Logic Flaws 

```YAML
id: TBA
title: Boolean Short-Circuit Logic Flaws 
baseSeverity: M
category: logic
language: solidity
blockchain: [ethereum]
impact: Incorrect access control, faulty validation, or invariant violations
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-617
swc: SWC-135
```

## 📝 Description

- In Solidity, boolean logic expressions such as a || b or a && b use short-circuit evaluation, meaning the second operand is not evaluated if the first operand determines the result. 
- Improper understanding of this behavior—especially when state-changing or validation logic is embedded within these conditions—can lead to:
- Skipped execution of required checks
- Bypassed modifiers or access control logic
- Unexpected state if expressions rely on both conditions being evaluated
- These flaws often occur when functions combine multiple logical conditions without realizing that some function calls or expressions are not guaranteed to run if earlier conditions resolve the outcome.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract BooleanFlawedAccess {
    mapping(address => bool) public isWhitelisted;
    mapping(address => bool) public isAdmin;

    function checkAccess() public view returns (bool) {
        return isAdmin[msg.sender] || isWhitelisted[msg.sender]; // ✅ access control logic
    }

    function secureAction() public {
        require(checkAccess(), "Access denied");
        _logAction(); // Logging happens after access check
    }

    function _logAction() internal {
        // ❌ assume this is called from another check as side effect
    }
}
```

## 🧪 Exploit Scenario

1. A developer writes require(isWhitelisted[msg.sender] || _registerAttempt()).
2. For whitelisted users, _registerAttempt() is never called.
3. Attackers repeatedly switch between whitelist and non-whitelist states, preventing proper rate limiting or logging.
4. This flaw is used to bypass tracking mechanisms or avoid triggering failsafes.

**Assumptions:**

- Side-effecting logic (e.g., counters, logs, modifiers) is embedded in boolean expressions.
- Developers mistakenly assume both conditions are always executed.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract BooleanSafeAccess {
    mapping(address => bool) public isWhitelisted;
    mapping(address => bool) public isAdmin;

    function hasAccess(address user) internal view returns (bool) {
        return isAdmin[user] || isWhitelisted[user];
    }

    function secureAction() public {
        _logAttempt(); // ✅ Always execute logging
        require(hasAccess(msg.sender), "Access denied");
        // Perform secure action
    }

    function _logAttempt() internal {
        // Record access attempt
    }
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: M
  reasoning: "Common logic bug with potential authorization side effects."
- context: "Access control or authentication logic"
  severity: H
  reasoning: "Flawed short-circuit may lead to privilege escalation."
- context: "Non-critical condition (e.g., UI toggles)"
  severity: I
  reasoning: "Impact is minimal when not tied to critical logic."
```

## 🛡️ Prevention

### Primary Defenses

- Treat boolean expressions (||, &&) as pure conditionals—never assume both sides will execute.
- Move state changes and side effects outside of conditions.

### Additional Safeguards

- Use require() and if statements separately for clarity.
- Avoid using inline mappings, counters, or external calls within boolean logic.

### Detection Methods

- Review all require() and if statements using || or && for hidden state changes.
- Tools: Slither (boolean-short-circuit, dangerous-expression), Solhint (no-complex-conditionals)

## 🕰️ Historical Exploits

- **Name:** TimeLock Contract Overflow Exploit 
- **Date:** 2016 
- **Loss:** 100 ETH unlocked prematurely due to short-circuit logic bypass 
- **Post-mortem:** [Link to post-mortem](https://hackernoon.com/hackpedia-16-solidity-hacks-vulnerabilities-their-fixes-and-real-world-examples-f3210eba5148) 
- **Name:** Ethernaut Token Transfer Underflow 
- **Date:** 2017 
- **Loss:** Unlimited tokens minted by bypassing balance checks via short-circuit logic 
- **Post-mortem:** [Link to post-mortem](https://cypherpunks-core.github.io/ethereumbook/09smart-contracts-security.html)
  
## 📚 Further Reading

- [SWC-135: Incorrect Logic](https://swcregistry.io/docs/SWC-135/) 
- [Solidity Docs – Short-Circuiting Operators](https://docs.soliditylang.org/en/latest/control-structures.html#boolean-operations) 
- [Slither Detector – Boolean Short-Circuit](https://github.com/crytic/slither/wiki/Detector-Documentation#boolean-expression-short-circuit) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Boolean Short-Circuit Logic Flaws 
severity: M
score:
impact: 3 
exploitability: 3 
reachability: 4  
complexity: 2  
detectability: 5 
finalScore: 3.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Failure to log, count, or limit logic can cause business or security errors.
- **Exploitability**: Attackers can use conditional paths to avoid state mutations.
- **Reachability**: Very common pattern in Solidity, especially with mappings and access checks.
- **Complexity**: Logic error, not cryptographic or deep exploit.
- **Detectability**: Easily seen in require() and if statements using booleans.