# Approval Reset Race Condition

```YAML
id: LS07M
title: Approval Reset Race Condition 
baseSeverity: M
category: token-approval
language: solidity
blockchain: [ethereum]
impact: Spender can use both old and new allowances before reset takes effect
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-362
swc: SWC-114
```

## 📝 Description

- Approval reset race condition occurs when a user attempts to update an ERC-20 allowance for a spender without first resetting it to zero. 
- This creates a potential race condition in which:
- The approved spender can front-run the allowance change,
- Use the existing allowance in a transaction,
- Then still be approved under the new allowance amount.
- This is a well-known flaw in the ERC-20 `approve()` design that can lead to unexpected token transfers, double-spending, or overspending if malicious actors exploit the timing.

## 🚨 Vulnerable Code

```solidity
function updateApproval(IERC20 token, address spender, uint256 newAmount) external {
    token.approve(spender, newAmount); // ❌ Does not reset to zero first
}
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. Alice has approved Bob for 100 tokens.
2. Alice wants to change Bob’s allowance to 50 and calls approve(spender, 50).
3. Bob sees the pending transaction and quickly calls transferFrom(alice, bob, 100) using the old allowance.

**Assumptions:**

- The token follows the ERC-20 standard without built-in increaseAllowance() and decreaseAllowance().
- The user calls approve() directly with a new non-zero value.

## ✅ Fixed Code

```solidity

function safeUpdateApproval(IERC20 token, address spender, uint256 newAmount) external {
    require(token.approve(spender, 0), "Reset failed");           // ✅ Step 1: reset to 0
    require(token.approve(spender, newAmount), "Update failed");  // ✅ Step 2: set new value
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: M
  reasoning: "Standard ERC20 vulnerability if not handled explicitly."
- context: "Widely-used token in external DeFi ecosystem"
  severity: H
  reasoning: "High impact due to scale and likelihood of malicious bots exploiting mempool."
- context: "ERC20 token with `increaseAllowance()`/`decreaseAllowance()`"
  severity: L
  reasoning: "Safer interfaces mitigate the race window risk."
```
## 🛡️ Prevention

### Primary Defenses

- Always reset allowance to zero before updating to a new non-zero value.
- Use increaseAllowance() and decreaseAllowance() instead of approve() directly.
- Use ERC-20 extensions or wrappers like SafeERC20 that implement these protections.

### Additional Safeguards

- Integrate frontend logic to warn users if allowance is being updated directly.
- Monitor for unexpected transferFrom() usage in transaction mempool during allowance updates.

### Detection Methods

- Slither: erc20-approval-race, unsafe-approve, approve-without-reset detectors.
- Static review of contracts interacting with ERC-20 tokens using approve() directly.
- Integration testing with transaction frontrunning simulations.

## 🕰️ Historical Exploits

- **Name:** ERC-20 Race Condition Disclosure 
- **Date:** 2018 
- **Impact:** Official ERC-20 known issue; mitigated in later libraries 
- **Post-mortem:** [Link to post-mortem](https://github.com/ethereum/EIPs/issues/20) 


## 📚 Further Reading

- [SWC-114: Unrestricted Token Approval](https://swcregistry.io/docs/SWC-114) 
- [Solidity Docs – ERC20 Approve Warning](https://docs.soliditylang.org/en/latest/) 
- [OpenZeppelin SafeERC20 Guide](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20) 
- [Slither ERC-20 Approval Checkers](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: LS07M
title: Approval Reset Race Condition 
severity: M
score:
impact: 4         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 3.75

```

---

## 📄 Justifications & Analysis

- **Impact**: High — allows spenders to use more than intended.
- **Exploitability**: Medium — depends on timing but very feasible in mempool-aware environments.
- **Reachability**: Common in DeFi apps and custom integrations.
- **Complexity**: Simple fix with minimal overhead.
- **Detectability**: High — easily caught in audits or with tools like Slither.