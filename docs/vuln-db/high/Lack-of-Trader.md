# Lack of Trader Signature Validation

```YAML
id: TBA
title: Lack of Trader Signature Validation
severity: H
category: authentication
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: Unauthorized trade, front-running, or user fund manipulation
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-306
swc: SWC-131
```

## 📝 Description

- In DeFi protocols where users can authorize off-chain trade intents, orders, or meta-transactions, failure to validate a trader’s digital signature before executing such actions exposes the system to:
- Front-running, where malicious actors execute trades meant for others
- Trade spoofing, where fake users are impersonated
- Order hijacking, where attackers profit from stolen or unvalidated intents
- If the contract allows public calls to executeTrade(), fillOrder(), or similar functions without requiring a valid cryptographic signature, any actor can trigger critical logic on behalf of others — leading to severe financial loss and protocol corruption.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract TradingPlatform {
    struct TradeIntent {
        address trader;
        address tokenIn;
        address tokenOut;
        uint256 amountIn;
        uint256 minOut;
        uint256 nonce;
    }

    mapping(address => uint256) public nonces;

    function executeTrade(TradeIntent calldata intent) external {
        require(intent.nonce == nonces[intent.trader], "Invalid nonce");
        nonces[intent.trader]++;

        // ❌ Missing: signature check to confirm msg.sender == trader
        // Logic executes trade on behalf of `intent.trader`
        _swap(intent.tokenIn, intent.tokenOut, intent.amountIn, intent.minOut, intent.trader);
    }

    function _swap(...) internal {
        // omitted
    }
}
```

## 🧪 Exploit Scenario

1. Alice signs a trade intent off-chain but never submits it.
2. Bob sees the signed intent in a public message (or worse, it's unsigned).
3. Bob submits the same trade (or crafts an intent) without Alice’s approval.
4. Bob manipulates tokenOut, minOut, or destination — profiting from Alice’s account or draining her balances.

**Assumptions:**

- No ecrecover() or EIP-712 signature check is enforced
- User intent is implicitly trusted without cryptographic proof

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract SecureTrading {
    using ECDSA for bytes32;

    struct TradeIntent {
        address trader;
        address tokenIn;
        address tokenOut;
        uint256 amountIn;
        uint256 minOut;
        uint256 nonce;
    }

    mapping(address => uint256) public nonces;

    function executeTrade(TradeIntent calldata intent, bytes calldata signature) external {
        require(intent.nonce == nonces[intent.trader], "Invalid nonce");

        bytes32 messageHash = keccak256(abi.encodePacked(
            intent.trader,
            intent.tokenIn,
            intent.tokenOut,
            intent.amountIn,
            intent.minOut,
            intent.nonce
        )).toEthSignedMessageHash();

        require(messageHash.recover(signature) == intent.trader, "Invalid signature");

        nonces[intent.trader]++;

        _swap(intent.tokenIn, intent.tokenOut, intent.amountIn, intent.minOut, intent.trader);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Validate all trade or action intents with ECDSA or EIP-712 signatures
- Bind signature to a unique nonce and optional expiry

### Additional Safeguards

- Include domain separator in hash (for replay protection across chains)
- Only allow execution of a user’s intent by themselves or approved relayers

### Detection Methods

- Identify any public function that takes user-addressed data (orders, trades) without signature checks
- Tools: Slither (missing-auth, tx.origin), manual logic review

## 🕰️ Historical Exploits

- **Name:** 0x Protocol Order Injection via Missing Signer Check
- **Date:** 2020-07 
- **Loss:** N/A (discovered during audit, patched pre-deployment)
- **Post-mortem:** [Link to post-mortem](https://0x.org/) 
  
## 📚 Further Reading

- [SWC-131: Missing Authentication](https://swcregistry.io/docs/SWC-131/) 
- [CWE-306: Missing Authentication for Critical Function](https://cwe.mitre.org/data/definitions/306.html) 
- [OpenZeppelin ECDSA Library](https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA) 
- [EIP-712 – Typed Structured Data Hashing and Signing](https://eips.ethereum.org/EIPS/eip-712)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Lack of Trader Signature Validation
severity: H
score:
impact: 5    
exploitability: 4 
reachability: 5  
complexity: 2    
detectability: 4  
finalScore: 4.35
```

---

## 📄 Justifications & Analysis

- **Impact**: A malicious actor can steal funds or impersonate users
- **Exploitability**: Simple exploit path if no signature is required
- **Reachability**: Found in almost all off-chain intent-based trading systems
- **Complexity**: Requires little sophistication — signature bypass only
- **Detectability**: Signature-less trade paths are easily flagged in audits