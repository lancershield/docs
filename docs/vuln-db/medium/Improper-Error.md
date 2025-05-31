# Improper Error Handling

```YAML
id: LS04M
title: Improper Error Handling  
baseSeverity: M
category: error-handling
language: solidity
blockchain: [ethereum]
impact: Critical operations may silently fail or proceed in an invalid state
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-391
swc: SWC-104
```

## 📝 Description

- Improper error handling in smart contracts refers to failing to check return values, suppressing reverts, or ignoring external call failures. 
- This can result in:
- Silent failures, where the user assumes an operation succeeded.
- Inconsistent state, where only part of a transaction executes.
- Security issues, like skipped permission checks or failed token transfers.
- Common mistakes include ignoring the return value of `transfer()`, `send()`, or `call()` and continuing execution without validating success.

## 🚨 Vulnerable Code

```solidity
contract ImproperErrorHandling {
    function withdraw(address payable to) public {
        // ❌ Ignores return value; failure won't revert
        to.send(1 ether);
        
        // Continues as if transfer succeeded...
        // Update state, emit event, etc.
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. User calls withdraw(), expecting to receive 1 ether.
2. The transfer fails (e.g., receiver is a contract that reverts or consumes too much gas).
3. Since the contract ignores the result of .send(), it proceeds as if the transfer succeeded.
4. The contract updates state or emits misleading events, leading to false reporting or fund loss.

**Assumptions:**

- No check on return value of .call(), .send(), or .transfer().
- Execution continues without revert or fallback.

## ✅ Fixed Code

```solidity

contract SafeErrorHandling {
    function withdraw(address payable to) public {
        (bool success, ) = to.call{value: 1 ether}("");
        require(success, "Transfer failed");
        // Safe to continue logic only if transfer succeeds
    }
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Commonly causes confusing failures, especially in DeFi protocols."
- context: "Critical protocol logic (staking, funds management)"
  severity: H
  reasoning: "Can lead to corrupted state or funds mismatch if failures are silently ignored."
- context: "Non-critical external call (logging or analytics)"
  severity: L
  reasoning: "Limited impact if only auxiliary behavior is affected."
```

## 🛡️ Prevention

### Primary Defenses

- Always check return values of external calls, especially .call, .send, and low-level operations.
- Use require(success, "...") to halt execution on failure.
- Avoid suppressing reverts with empty catch blocks unless absolutely necessary.

### Additional Safeguards

- Use OpenZeppelin wrappers (SafeERC20, Address.sendValue) for safe external interactions.
- Emit failure logs if operation must continue despite a failed call.
- Avoid complex inline try/catch flows unless explicitly handled.

### Detection Methods

- Slither: unchecked-return, ignored-call, error-handling detectors.
- Manual audit of external calls or token transfers.
- Unit tests that simulate failures (reverts, fallback gas exhaustion).

## 🕰️ Historical Exploits

- **Name:** Parity Multisig Wallet v1 
- **Date:** 2017 
- **Loss:** ~$30M frozen 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert/) 


## 📚 Further Reading

- [SWC-104: Unchecked Call Return Value](https://swcregistry.io/docs/SWC-104) 
- [OpenZeppelin – Address.sendValue()](https://docs.openzeppelin.com/contracts/4.x/api/utils#Address-sendValue-address-payable-uint256-) 
- [Solidity Docs – External Function Calls](https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions) 

---

## ✅ Vulnerability Report

```markdown
id: LS04M
title: Improper Error Handling 
severity: M
score:
impact: 3         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Can mislead users, silently block funds, or enable partial execution of dangerous logic.
- **Exploitability**: Requires no attacker sophistication—just a fallback revert or failed external call.
- **Reachability**: Seen in core finance logic: withdraw, buy, sell, approve, etc.
- **Complexity**: Low; relies on call return behavior.
- **Detectability**: Readily caught by Slither or during manual code review and testing.

