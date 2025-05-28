# Signature Malleability

```YAML
id: TBA
title: Signature Malleability 
baseSeverity: H
category: cryptography
language: solidity
blockchain: [ethereum]
impact: Signature reuse, replay attacks, and bypass of signature uniqueness checks
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-347
swc: SWC-117
```

## 📝 Description

- In Ethereum, ECDSA signatures are malleable: for a given valid signature (r, s, v), a second valid signature can be created by flipping the s value to its low-s/high-s complement (r, -s mod n, v ⊕ 1). If the contract does not enforce canonical form (low-s form), attackers may:
- Replay signatures across chains or contracts
- Bypass uniqueness checks in usedSignatures mappings or nonces
- Force inconsistent hashes or cache mismatches
- Invalidate domain-specific replay protection (e.g., EIP-712)

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract SignatureVerifier {
    mapping(bytes32 => bool) public usedSignatures;

    function verify(bytes32 messageHash, bytes memory signature) public {
        require(!usedSignatures[keccak256(signature)], "Signature used");

        address signer = recover(messageHash, signature);
        require(signer != address(0), "Invalid signature");

        usedSignatures[keccak256(signature)] = true; // ❌ vulnerable to malleable duplicates
    }

    function recover(bytes32 hash, bytes memory sig) public pure returns (address) {
        (bytes32 r, bytes32 s, uint8 v) = split(sig);
        return ecrecover(hash, v, r, s);
    }

    function split(bytes memory sig) internal pure returns (bytes32 r, bytes32 s, uint8 v) {
        assembly {
            r := mload(add(sig, 32))
            s := mload(add(sig, 64))
            v := byte(0, mload(add(sig, 96)))
        }
    }
}
```

## 🧪 Exploit Scenario

1. A user signs a message, producing (r, s, v) and submits it to the smart contract.
2. The attacker observes the signature on-chain.
3. They compute a second valid signature (r, n - s, 27 ⊕ 1) (same r, flipped s, flipped v).
4. Since the keccak256(signature) differs, the replay check passes.
5. The attacker calls verify() with the malleable version and executes the same logic again, bypassing protections.

**Assumptions:**

- No enforcement of low-s form or EIP-2 compliance.
- Signatures are validated using ecrecover or raw keccak checks.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeSignatureVerifier {
    mapping(bytes32 => bool) public usedMessages;

    function verify(bytes32 messageHash, bytes memory signature) public {
        address signer = recoverSafe(messageHash, signature);
        require(signer != address(0), "Invalid signature");

        bytes32 key = keccak256(abi.encodePacked(signer, messageHash));
        require(!usedMessages[key], "Signature used");

        usedMessages[key] = true;
    }

    function recoverSafe(bytes32 hash, bytes memory sig) public pure returns (address) {
        (bytes32 r, bytes32 s, uint8 v) = split(sig);
        require(uint256(s) <= 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF, "Non-canonical s"); // ✅ enforce low-s
        return ecrecover(hash, v, r, s);
    }

    function split(bytes memory sig) internal pure returns (bytes32 r, bytes32 s, uint8 v) {
        require(sig.length == 65, "Bad signature length");
        assembly {
            r := mload(add(sig, 32))
            s := mload(add(sig, 64))
            v := byte(0, mload(add(sig, 96)))
        }
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Replay or unauthorized execution risk in systems relying on signature uniqueness."
- context: "Cross-chain permit system without chainId binding"
  severity: C
  reasoning: "Critical systemic exploit risk due to chain replay possibility."
- context: "Closed system with enforced nonces and low s checks"
  severity: L
  reasoning: "Proper protections mitigate most exploit paths."
```

## 🛡️ Prevention

### Primary Defenses

- Reject non-canonical (high-s) signatures.
- Use OpenZeppelin’s ECDSA.recover() or equivalent libraries that enforce EIP-2 compliance.

### Additional Safeguards

- Never rely on keccak256(signature) alone for uniqueness; use signer + message hash.
- Apply domain separation (e.g., EIP-712) to prevent cross-protocol replay.

### Detection Methods

- Search for ecrecover() and check whether s-value bounds are enforced.
- Tools: Slither (signature-malleability), Semgrep, MythX

## 🕰️ Historical Exploits

- **Name:** Mt. Gox Bitcoin Exchange Collapse 
- **Date:** 2014 
- **Loss:** Approximately 850,000 BTC 
- **Post-mortem:** [Link to post-mortem](https://arxiv.org/abs/1403.6676) 
- **Name:** Ethereum Parity Wallet Hack 
- **Date:** 2017 
- **Loss:** Approximately $30 million 
- **Post-mortem:** [Link to post-mortem](https://metana.io/blog/how-do-signatures-and-malleability-impact-web3-security/) 
  
## 📚 Further Reading

- [SWC-117: Signature Malleability](https://swcregistry.io/docs/SWC-117/) 
- [Solidity Docs – ecrecover](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#mathematical-and-cryptographic-functions) 
- [OpenZeppelin ECDSA Library](https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA) 
- [EIP-2: Homestead Changes](https://eips.ethereum.org/EIPS/eip-2)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Signature Malleability 
severity: H
score:
impact: 4  
exploitability: 4 
reachability: 4  
complexity: 2    
detectability: 4 
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Replay or duplication of transactions/actions with alternate signature forms.
- **Exploitability**: Well-documented and scriptable with existing tooling.
- **Reachability**: Present in most signature-based auth schemes if not mitigated.
- **Complexity**: Non-obvious unless developer understands ECDSA math.
- **Detectability**: Visible in signature recovery code without low-s enforcement.