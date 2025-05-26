#  ABI encodePacked Collision

```YAML
id: TBA
title: ABI encodePacked Collision 
baseSeverity: H
category: encoding
language: solidity
blockchain: [ethereum]
impact: Signature spoofing, hash collision, or critical logic bypass
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-180
swc: SWC-136
```

## 📝 Description

- abi.encodePacked() is a low-level encoding function that returns a tightly packed byte array without padding. 
- While this is gas-efficient and useful in some scenarios (e.g., keccak256 hashing), it can introduce dangerous collisions when used with variable-length arguments such as strings, bytes, or uint concatenated with user-controlled input.
- A classic example: keccak256(abi.encodePacked(a, b)) may result in the same output for two different combinations of (a, b) due to tight packing, especially when types like string, bytes, and uint are concatenated without separators or fixed lengths.
- When used in signature verification, identity hashing, or uniqueness constraints, this can lead to:
- Forged signatures
- Identity collisions
- Front-running or replay attacks

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract CollidingHasher {
    mapping(bytes32 => bool) public used;

    function markUsed(string memory prefix, uint256 id) public {
        bytes32 hash = keccak256(abi.encodePacked(prefix, id));
        require(!used[hash], "Already used");
        used[hash] = true;
    }
}
```

## 🧪 Exploit Scenario

1. Contract generates a unique hash using abi.encodePacked(userInput, amount) for anti-replay or uniqueness.
2. Attacker finds multiple combinations of input parameters that result in the same keccak256 hash.
3. They reuse or spoof a previously valid hash to bypass require(!used[hash]).
4. This allows unauthorized actions, such as double minting, duplicated identity claims, or replayed off-chain signatures.

**Assumptions:**

- The contract uses encodePacked + keccak256 to construct unique identifiers or signed messages.
- The input includes dynamic-length or ambiguous types (string, bytes, uint, etc.).
- No delimiter or structured type separation is enforced.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeHasher {
    mapping(bytes32 => bool) public used;

    function markUsed(string memory prefix, uint256 id) public {
        bytes32 hash = keccak256(abi.encode(prefix, id)); // ✅ Use abi.encode with padding
        require(!used[hash], "Already used");
        used[hash] = true;
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Hash used in signature validation, claim ID, or unique key"
  severity: H
  reasoning: "Hash collision allows spoofing, multi-claiming, or key overwrites."
- context: "Hash used for low-risk UI or analytics"
  severity: L
  reasoning: "Impact minimal if not tied to security or fund movement."
- context: "abi.encode() or hashed single arguments only"
  severity: I
  reasoning: "Collision risk negligible—well-structured input."
```

## 🛡️ Prevention

### Primary Defenses

- Use abi.encode() for all signature, hashing, and uniqueness purposes.
- Avoid abi.encodePacked() when dealing with multiple dynamic or user-controlled inputs.

### Additional Safeguards

- Add delimiters or length prefixes when using encodePacked.
- Normalize input types or use struct encoding where possible.
- Validate all parameters thoroughly when decoding signed data.

### Detection Methods

- Static analysis to flag abi.encodePacked() inside keccak256(...) or signature generators.
- Review hashing schemes involving variable user inputs.
- Tools: Slither (dangerous-packed-encoding), MythX, manual audit

## 🕰️ Historical Exploits

- **Name:** Poly Network Exploit
- **Date:** 2021-08-10
- **Loss:** $610 million
- **Post-mortem:** [Link to post-mortem](https://blog.gopluslabs.io/vulnerabilities-cases/smart-contract/hash-collision/poly-network)

## 📚 Further Reading

- [SWC-136: Unencrypted Sensitive Data](https://swcregistry.io/docs/SWC-136/) 
- [Understanding Hash Collisions: abi.encodePacked in Solidity – Nethermind](https://www.nethermind.io/blog/understanding-hash-collisions-abi-encodepacked-in-solidity) 
- [The trap of using encodePacked in Solidity – OpenZeppelin Forum](https://forum.openzeppelin.com/t/the-trap-of-using-encodepacked-in-solidity/1052)
   
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: ABI encodePacked Collision 
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 3   
complexity: 2     
detectability: 4  
finalScore: 3.75
```

---

## 📄 Justifications & Analysis

- **Impact**: Can break uniqueness guarantees, signature schemes, and lead to fund duplication or misuse.
- **Exploitability**: Attackers only need to identify a collision pair to exploit weak encodePacked usage.
- **Reachability**: Hashes and signatures are typically user-input dependent and publicly callable.
- **Complexity**: Low complexity exploit, but requires input crafting or brute-force collisions.
- **Detectability**: Subtle, especially when developers use encodePacked for optimization without recognizing ambiguity.







