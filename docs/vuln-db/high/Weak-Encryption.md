# Weak Encryption

```YAML
id: TBA
title: Weak Encryption
severity: H
category: cryptography
language: solidity
blockchain: [ethereum, bsc, polygon, arbitrum, optimism]
impact: Sensitive data or access controls can be reverse-engineered
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-326: Inadequate Encryption Strength
swc: SWC-136: Unrestricted Critical Variable Update (if relevant), SWC-133: Hash Collisions
```

## 📝 Description

- Smart contracts are deterministic and transparent. Using weak cryptographic primitives—or using strong ones incorrectly—exposes sensitive logic, access conditions, or secrets to on-chain brute-force or offline attacks.
- Simple XOR-based obfuscation
- Hardcoded secrets encoded via keccak256(abi.encodePacked(...))
- Low-entropy challenge-response schemes (e.g., math puzzles, small integers)
- Using block.timestamp or predictable values as keys or salts
- Unlike traditional backend environments, on-chain data (including calldata and storage) is public. Thus, any “encryption” relying on obscurity or weak randomness can be reverse-engineered, exposing critical logic.

## 🚨 Vulnerable Code

```solidity

bytes32 public passwordHash = keccak256(abi.encodePacked("secret123"));

function unlock(string memory password) public {
    require(keccak256(abi.encodePacked(password)) == passwordHash, "Wrong password");
    // 🔓 grant access
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. The contract stores passwordHash = keccak256("secret123") as a public variable.
2. An attacker downloads the bytecode and finds the hash value.
3. Using a hash cracker (e.g., Hashcat or custom script), they iterate over common passwords.
4. secret123 is recovered as the preimage.
5. The attacker calls unlock("secret123") and gains unauthorized access to admin or withdrawal functionality.

**Assumptions:**

- Hash used as authentication without sufficient entropy.
- No rate-limiting or one-time-use mechanism.
- Entire hash and unlock logic is visible on-chain.

## ✅ Fixed Code

```solidity

function unlock(bytes32 messageHash, bytes memory signature) public {
    address signer = recoverSigner(messageHash, signature);
    require(signer == authorizedSigner, "Unauthorized");
    // grant access
}
```

## 🛡️ Prevention

### Primary Defenses

- Never rely on on-chain hashes or reversible encryption for sensitive logic.
- Use off-chain authorization (e.g., signatures, Merkle proofs).
- Use cryptographic challenges with high entropy (e.g., 128-bit+ secrets or randomness).

### Additional Safeguards

- Avoid publishing preimages even in event logs.
- Monitor for hash preimage exposure or dictionary attack vectors.

### Detection Methods

- Static analysis for low-entropy secrets or public hash comparisons.
- Manual review of keccak256(...) == hash or xor(...) constructs.
- Tools: Slither (low-entropy plugins), MythX entropy analyzer, custom scripts

## 🕰️ Historical Exploits

- **Name:** Gatekeeper Challenge (CTF) 
- **Date:** 2021 
- **Loss:** N/A  
- **Post-mortem:** [Link to post-mortem](https://ethernaut.openzeppelin.com/level/13)

## 📚 Further Reading

- [SWC-136: Unrestricted Critical Variable Update](https://swcregistry.io/docs/SWC-136) 
- [Ethereum StackExchange – Why On-Chain Secrets Are Unsafe](https://ethereum.stackexchange.com/questions/8514) 
- [OpenZeppelin ECDSA Signatures](https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA) 
 
---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Weak Encryption
severity: H
score:
impact: 4  
exploitability: 3   
reachability: 4 
complexity: 3  
detectability: 4  
finalScore: 3.7
```

---

## 📄 Justifications & Analysis

- **Impact**: Can lead to full access control bypass if weak encryption gates critical logic.
- **Exploitability**: Feasible if entropy is low or hardcoded; off-chain brute-force can recover preimages.
- **Reachability**: Common in simple puzzle contracts, games, or naive access control implementations.
- **Complexity**: Requires some understanding of hash cracking, but tools are widely available.
- **Detectability**: Static analysis, Slither, and code review can reveal these patterns easily.

