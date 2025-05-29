# Empty Catch Blocks

```YAML
id: TBA
title: Empty Catch Blocks
baseSeverity: L
category: error-handling
language: solidity
blockchain: [ethereum]
impact: Silent failure of important logic
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.6.0", "<latest"]
cwe: CWE-391
swc: SWC-134
```
## 📝 Description

- Since Solidity v0.6.0, try/catch blocks can be used to handle failures from external function calls and contract creation.
- However, if the catch block is left empty, it silently suppresses all errors—including failed delegatecalls, low-level reverts, and out-of-gas errors.
- This leads to:
- Loss of visibility into critical failures
- Broken invariants when expected outcomes are skipped
- Security risks when failed calls should halt execution (e.g., reentrancy guards, token transfers)
- Empty catch blocks create misleading test results and undermine auditability, especially in permissioned logic like upgradability, bridging, or reward claims.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IStrategy {
    function harvest() external;
}

contract Vault {
    IStrategy public strategy;

    function runHarvest() external {
        try strategy.harvest() {
            // success
        } catch {
            // ❌ silently ignore all errors
        }
    }
}
```

## 🧪 Exploit Scenario

1. A vault attempts to call a strategy's harvest() via try/catch.
2. Due to an internal error (e.g., insufficient balance or bad state), harvest() fails.
3. The catch block is empty, so nothing is logged or acted upon.
4. Rewards are never distributed, and users experience unexplained reward halts.
5. In an attacker-crafted call, this behavior could be exploited to bypass downstream logic that relies on harvest success.

**Assumptions:**

- Errors are non-trivial and affect system flow.
- Empty catch does not trigger fallback behavior or alerts.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeVault {
    IStrategy public strategy;

    event HarvestFailed(address strategy, string reason);

    function runHarvest() external {
        try strategy.harvest() {
            // success
        } catch Error(string memory reason) {
            emit HarvestFailed(address(strategy), reason);
        } catch {
            emit HarvestFailed(address(strategy), "Unknown error");
        }
    }
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: L
  reasoning: "May reduce observability but doesn’t break core logic."
- context: "Protocol with critical external calls"
  severity: M
  reasoning: "Can hide serious errors in price feeds, upgrades, or transfers."
- context: "Development-phase contracts with logging enabled"
  severity: I
  reasoning: "Low severity if debugging tools or logs capture failures elsewhere."
```

## 🛡️ Prevention

### Primary Defenses

- Always log or handle the error in catch blocks.
- Never leave catch {} empty—emit diagnostics or revert conditionally.

### Additional Safeguards

- Use require(success) or guard post-conditions when relying on side effects.
- Monitor HarvestFailed/ExternalCallFailed events with off-chain observability tools.

### Detection Methods

- Search for catch {} with empty body.
- Use linters or static analysis to flag unhandled exceptions.
- Tools: Slither (unhandled-exceptions), Solhint (no-empty-blocks)

## 🕰️ Historical Exploits

- **Name:** Empty Catch Blocks Leading to Silent Failures
- **Date:** Various 
- **Loss:** Potential for hidden bugs, security issues, and unreliable code due to unhandled exceptions 
- **Post-mortem:** [Link to post-mortem](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/empty-catch.html) -
  
## 📚 Further Reading

- [SWC-134: Unhandled Exceptions](https://swcregistry.io/docs/SWC-134/) 
- [Solidity Docs – Try/Catch](https://docs.soliditylang.org/en/latest/control-structures.html#try-catch)  
- [Trail of Bits: Solidity Anti-Patterns](https://blog.trailofbits.com/) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Empty catch Blocks 
severity: L
score:
impact: 3 
exploitability: 2 
reachability: 4  
complexity: 1  
detectability: 5
finalScore: 3.
```

---

## 📄 Justifications & Analysis

- **Impact**: Failures are hidden from users and developers, allowing silent failure loops.
- **Exploitability**: Triggering errors can cause desired failure paths to be skipped.
- **Reachability**: Appears in many upgradeable systems, strategies, and oracle consumers.
- **Complexity**: Easy to fix—requires a log or guard inside the catch.
- **Detectability**: Very easy to catch with static tools or basic reviews.

