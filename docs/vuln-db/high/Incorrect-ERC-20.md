# Incorrect ERC-20 Interface Causes Token Transfer Failures 

```YAML
id: TBA
title: Incorrect ERC-20 Interface Causes Token Transfer Failures or Fund Loss
severity: H
category: interface-mismatch
language: solidity
blockchain: [ethereum]
impact: Funds lost due to false success assumptions or silent reverts
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-704
swc: SWC-135
```

## 📝 Description

- The ERC-20 standard defines transfer, approve, and transferFrom functions, but early token implementations (e.g., USDT, BNB) deviate from the spec by:
- Not returning a bool on success/failure
- Reverting silently instead of returning false
- If a contract expects all ERC-20 tokens to return bool and ignores the return value, it may:
- Assume success when no transfer occurred
- Miss silent reverts or no-op behavior
- Proceed with incorrect state (e.g., issuing a receipt without the token transfer)
- This interface mismatch can result in fund loss, reward inflation, or stuck tokens.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IERC20Broken {
    function transfer(address to, uint256 amount) external; // ❌ No return value
}

contract RewardPool {
    function payout(IERC20Broken token, address recipient, uint256 amount) external {
        token.transfer(recipient, amount); // ❌ No return value checked
    }
}
```

## 🧪 Exploit Scenario

1. Contract uses IERC20.transfer() assuming all tokens return a bool.
2. Token like USDT or BNB (which do not return a value) is passed in.
3. transfer() executes, returns nothing — Solidity fills returndata with zero or garbage.
4. Contract assumes success; updates accounting or emits events.
5. Users receive rewards or access without receiving actual tokens.

**Assumptions:**

- Developer assumes compliance with ERC-20.
- Token passed in behaves non-standardly (e.g., no return value, always reverts).

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

interface IERC20 {
    function transfer(address to, uint256 amount) external returns (bool);
}

library SafeERC20 {
    function safeTransfer(IERC20 token, address to, uint256 value) internal {
        (bool success, bytes memory data) = address(token).call(
            abi.encodeWithSelector(token.transfer.selector, to, value)
        );
        require(success && (data.length == 0 || abi.decode(data, (bool))), "Transfer failed");
    }
}

contract RewardPool {
    function payout(IERC20 token, address recipient, uint256 amount) external {
        SafeERC20.safeTransfer(token, recipient, amount); // ✅ Handles all cases
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use OpenZeppelin's SafeERC20 for all token interactions.
- Never ignore return values from transfer, approve, or transferFrom.

### Additional Safeguards

- Whitelist tested tokens with consistent ERC-20 behavior.
- Consider custom wrappers for known non-standard tokens (e.g., USDTWrapper).

### Detection Methods

- Search for ignored return values from transfer, approve, or transferFrom.
- Audit ERC-20 interfaces and compare to live deployed token ABIs.
- Tools: Slither (unchecked-transfer), MythX, custom linters

## 🕰️ Historical Exploits

- **Name:** Compound cToken Transfer Bug 
- **Date:** 2020 
- **Loss:** ~$0.5M loss potential (mitigated) 
- **Post-mortem:** [Link to post-mortem](https://compound.finance/docs/comp#compound-governance-bug) 

## 📚 Further Reading

- [SWC-135: Incorrect Authorization](https://swcregistry.io/docs/SWC-135/) 
- [OpenZeppelin – SafeERC20 Documentation](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20)
- [Solidity Docs – External Calls and Return Values](https://docs.soliditylang.org/en/latest/control-structures.html#external-function-calls) 
- [USDT Token Transfer Quirk Analysis](https://ethereum.stackexchange.com/questions/39352/does-the-tether-usdt-token-contract-implement-the-erc20-spec) 

---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Incorrect ERC-20 Interface Causes Token Transfer Failures or Fund Loss
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 4   
complexity: 2 
detectability: 4  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Silent transfer failures undermine asset movement and trust in the protocol.
- **Exploitability**: Easy if attackers use or recommend incompatible tokens.
- **Reachability**: ERC-20 interactions exist in nearly all DeFi contracts.
- **Complexity**: Simple misuse of interface; misunderstanding spec is common.
- **Detectability**: Missed unless SafeERC20 is enforced or tests cover non-standard tokens.