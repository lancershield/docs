# Non-Reverting Functions 

```YAML
id: TBA
title: Non-Reverting Functions 
baseSeverity: M
category: error-handling
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Unreliable execution, user fund loss, logic inconsistency
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-252
swc: SWC-123
```

## 📝 Description

- According to the ERC-20 standard, the approve() function should return a boolean indicating success. However, many real-world ERC-20 tokens are non-standard or broken, and either:
- Do not return a boolean at all
- Return garbage data or nothing
- Do not revert on failure, making it hard for calling contracts to detect errors
- If a contract interacts with such broken tokens assuming a valid true/false return, it may:
- Assume the approval succeeded when it did not
- Proceed with transfers using stale allowances
- Allow spoofed approval logic, where malicious tokens always return true

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IERC20 {
    function approve(address spender, uint256 amount) external returns (bool);
}

contract VulnerableVault {
    function approveToken(address token, address spender, uint256 amount) external {
        bool success = IERC20(token).approve(spender, amount); // ❌ assumes ERC20 compliance
        require(success, "Approval failed");
    }
}
```

## 🧪 Exploit Scenario

1. Alice uses a contract to interact with a non-standard token (e.g., USDT).
2. The token’s approve() does not return anything (missing returns (bool)).
3. The contract checks the return value and assumes the approval failed (false).
4. Transaction reverts, blocking functionality.
5. In some cases, attacker-controlled tokens could always return true, bypassing allowance checks in wrapped logic.

**Assumptions:**

- The token is non-standard and behaves differently than OpenZeppelin’s ERC20
- The dApp or vault assumes approve() returns a bool and is compliant

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract SafeVault {
    using SafeERC20 for IERC20;

    function approveToken(address token, address spender, uint256 amount) external {
        IERC20(token).safeApprove(spender, amount); // ✅ handles non-standard tokens
    }
}
```
## 🧭 Contextual Severity

```yaml
- context: "Financial logic relying on unchecked return values"
  severity: M
  reasoning: "Can lead to partial failures, fund loss, and broken state"
- context: "Low-risk view or logging function"
  severity: I
  reasoning: "No financial or state impact"
- context: "Function fully protected with `require()` and SafeERC20"
  severity: I
  reasoning: "Vulnerability mitigated"
```
## 🛡️ Prevention

### Primary Defenses

- Always use SafeERC20.safeApprove(), safeTransfer(), and safeTransferFrom()
- Reject interacting with known broken tokens unless explicitly handled

### Additional Safeguards

- Maintain allowlists/deny-lists for known broken ERC20 tokens
- Detect and log any token interaction inconsistencies

### Detection Methods

- Check for interfaces that assume approve() returns a bool
- Tools: Slither (erc20-interface-violations), MythX, ConsenSys Diligence linter

## 🕰️ Historical Exploits

- **Name:** OMG Token Transfer Compatibility Failures 
- **Date:** 2021-07 
- **Loss:** ~$80,000 
- **Post-mortem:** [Link to post-mortem](https://github.com/crytic/slither/wiki/Detector-Documentation#erc20-interface-violations)
  
## 📚 Further Reading

- [ERC-20 Standard](https://eips.ethereum.org/EIPS/eip-20)
- [OpenZeppelin SafeERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20)
- [Slither – Unsafe ERC20 Usage Detection](https://github.com/crytic/slither/wiki/Detector-Documentation#erc20-interface-violations)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Non-Reverting Functions
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

- **Impact**: Can cause critical logic assumptions to fail (e.g., spend limits, access denial)
- **Exploitability**: Easily triggered with common non-compliant tokens like USDT or custom malicious tokens
- **Reachability**: Extremely common in any contract integrating ERC20s
- **Complexity**: Relies on understanding quirks in ERC20 implementations
- **Detectability**: Well-known and detectable through static analysis or testing