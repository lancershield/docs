# Ignoring SafeERC20

```YAML
id: TBA
title: Ignoring SafeERC20
baseSeverity: H
category: token-integration
language: solidity
blockchain: [ethereum]
impact: Lost tokens or stuck state due to non-standard ERC20 behavior
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.24"]
cwe: CWE-754
swc: SWC-135
```
## 📝 Description

- Ignoring SafeERC20 refers to using raw `IERC20.transfer()` or `IERC20.transferFrom()` calls directly without handling return values or using OpenZeppelin's SafeERC20 wrapper. While some tokens follow the ERC-20 standard strictly, others:
- Do not return `true`/`false` values, or
- Revert silently on failure, or
- Behave unexpectedly (e.g., not throwing on failure or returning nothing).
- This can result in:
- Silent transfer failures
- Funds not being sent or received as expected,
- Incorrect balance assumptions in subsequent logic.

## 🚨 Vulnerable Code

```solidity
contract UnsafeTokenTransfer {
    function deposit(IERC20 token, uint256 amount) external {
        // ❌ No return value checked, no SafeERC20 wrapper
        token.transferFrom(msg.sender, address(this), amount); 
    }

    function withdraw(IERC20 token, uint256 amount) external {
        token.transfer(msg.sender, amount); // May silently fail
    }
}
```

## 🧪 Exploit Scenario

Step-by-step impact:

1. Contract interacts with a non-standard ERC20 (e.g., USDT or BNB token).
2. transferFrom() fails internally but returns nothing (does not revert).
3. No error is thrown — logic continues as if tokens were transferred.
4. Contract state becomes desynchronized (e.g., credit given without funds received).

**Assumptions:**

- No wrapper or return value check is used.
- Token follows a non-standard ERC20 implementation (e.g., no return value or reverts internally).

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract SafeTokenTransfer {
    using SafeERC20 for IERC20;

    function deposit(IERC20 token, uint256 amount) external {
        token.safeTransferFrom(msg.sender, address(this), amount);
    }

    function withdraw(IERC20 token, uint256 amount) external {
        token.safeTransfer(msg.sender, amount);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Any ERC20 interaction may silently fail if token is non-compliant."
- context: "Well-audited DeFi protocol using only known standard tokens"
  severity: M
  reasoning: "Assumes audited compliance but remains vulnerable to edge tokens."
- context: "Protocol integrating user-supplied tokens (e.g., DEXs)"
  severity: C
  reasoning: "High probability of incompatible token interaction and lost funds."
```

## 🛡️ Prevention

### Primary Defenses

- Use OpenZeppelin’s SafeERC20 for all transfers and approvals.
- Avoid using raw IERC20.transfer() or transferFrom() directly.

### Additional Safeguards

- Validate that tokens used conform to the standard or explicitly support non-standard tokens via adapter wrappers.
- Include integration tests using popular non-compliant tokens (e.g., USDT, BUSD).
- Consider a try/catch fallback pattern for known edge-case tokens.

### Detection Methods

- Slither: erc20-transfer-no-check, unsafe-transfer, missing-safe-wrapper detectors.
- Manual code review of any IERC20 calls.
- Automated linting and CI rules to enforce SafeERC20 usage.

## 🕰️ Historical Exploits

- **Name:** Uniswap V2 Liquidity Migration Issue 
- **Date:** 2021 
- **Loss:** Imbalanced liquidity pools 
- **Post-mortem:** [Link to post-mortem](https://uniswap.org/blog/uniswap-v2/) 
  

## 📚 Further Reading

- [SWC-135: Code With No Effects or Incomplete Logic](https://swcregistry.io/docs/SWC-135) 
- [OpenZeppelin Docs – SafeERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20) 
- [USDT Transfer Quirks](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/343) 
- [Slither Detectors for Token Transfers](https://github.com/crytic/slither)

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Ignoring SafeERC20 
severity: H
score:
impact: 4        
exploitability: 2 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 3.4
```

---

## 📄 Justifications & Analysis

- **Impact**: Moderate to severe if tokens fail silently or revert internally.
- **Exploitability**: Limited — attacker must exploit non-standard behavior or frontend assumptions.
- **Reachability**: Very common across DEXs, farms, and vaults.
- **Complexity**: Simple fix using SafeERC20.
- **Detectability**: High — well-documented, and most static analyzers catch this.