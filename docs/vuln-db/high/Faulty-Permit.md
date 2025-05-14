# Faulty Permit Implementation Enabling Unauthorized Token Transfers


```YAML
id: TBA
title: Faulty Permit Implementation Enabling Unauthorized Token Transfers
severity: H
category: signature-verification
language: solidity
blockchain: [ethereum]
impact: Attacker bypasses approval flow and spends tokens without consent
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<0.8.21"]
cwe: CWE-347
swc: SWC-122
```

## 📝 Description

- Faulty permit implementations occur when ERC-2612-style `permit()` functions are implemented incorrectly, particularly in:
- Signature digest construction (EIP-712),
- `nonce` tracking,
- `domainSeparator` usage,
- Or validation logic.
- When `permit()` fails to properly validate the signed data or incorrectly handles replays or approvals, it enables:
- **Unauthorized token approvals**, or
- **Replay attacks**, letting attackers spend tokens repeatedly.

## 🚨 Vulnerable Code

```solidity
contract BrokenPermitToken {
    mapping(address => uint256) public nonces;

    function permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v, bytes32 r, bytes32 s
    ) external {
        // ❌ Missing deadline check
        bytes32 digest = keccak256(abi.encodePacked(owner, spender, value, nonces[owner]));
        address signer = ecrecover(digest, v, r, s);
        require(signer == owner, "Invalid signature");

        nonces[owner]++; // ✅ Used but digest was incorrectly formed

        // Assume this sets allowance (omitted for brevity)
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker sees a valid signature from a user that includes owner, spender, value, nonce.
2. Since no deadline is enforced, signature never expires.
3. Attacker reuses the same signature on multiple chains or contracts where nonces[owner] hasn't changed.
4. The faulty digest allows permit() to succeed again and again, allowing unlimited token approvals.

**Assumptions:**

- Permit digest is non-EIP-712 conformant or missing deadline.
- nonces[owner] is not enforced across domains or replay-safe contexts.

## ✅ Fixed Code

``` solidity

import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";

contract SafePermitToken is ERC20Permit {
    constructor(string memory name) ERC20(name, "SPT") ERC20Permit(name) {}

    // Uses OpenZeppelin's validated implementation of EIP-2612
}
```

## 🛡️ Prevention

### Primary Defenses

- Use OpenZeppelin’s ERC20Permit or draft-ERC20Permit contracts.
- Always enforce:
- require(block.timestamp <= deadline, "Permit expired");
- EIP-712-compliant domain separator
- Unique nonce per signer

### Additional Safeguards

- Use DOMAIN_SEPARATOR() with EIP712 helper contract to avoid inconsistencies.
- Validate all permit() calls using test vectors and tooling like eth-permit or ethers.utils._TypedDataEncoder.

### Detection Methods

- Slither: incorrect-ecrecover-digest, missing-deadline, signature-replay detectors.
- Manual inspection of digest construction and nonce handling.
- Unit tests using replayed signatures and expired deadlines.

## 🕰️ Historical Exploits

- **Name:** SushiSwap xSUSHI Permit Bypass 
- **Date:** 2021 
- **Loss:** N/A (mitigated pre-deployment) 
- **Post-mortem:** [Link](https://twitter.com/0xfoobar/status/1384228458057744385) 


## 📚 Further Reading

- [SWC-122: Signature Replay](https://swcregistry.io/docs/SWC-122) 
- [EIP-2612: permit()](https://eips.ethereum.org/EIPS/eip-2612) 
- [OpenZeppelin – ERC20Permit Docs](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit)
- [eth-permit tool](https://github.com/dmihal/eth-permit) 


---
## ✅ Vulnerability Report 


```markdown
id: TBA
title: Faulty Permit Implementation Enabling Unauthorized Token Transfers
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 4  
finalScore: 4.3
```


---

## 📄 Justifications & Analysis

- **Impact**: Attackers can grant themselves approvals, withdraw tokens, or abuse liquidity.
- **Exploitability**: Simple replay or signature injection possible with faulty digest.
- **Reachability**: Widespread due to adoption of EIP-2612.
- **Complexity**: Moderate — attacker needs to craft or reuse a valid signature.
- **Detectability**: Very detectable with audits, test vectors, and OpenZeppelin comparison.