# Merkle Proof Misuse

```YAML
id: TBA
title: Merkle Proof Misuse Allowing Unauthorized Claims 
severity: H
category: merkle-proof
language: solidity
blockchain: [ethereum]
impact: Unauthorized access to functions, funds, or features
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<0.8.21"]
cwe: CWE-347
swc: SWC-121
```

## 📝 Description

- Merkle Proof Misuse occurs when smart contracts use Merkle trees for whitelisting, access control, or claim validation but implement the verification logic incorrectly. This includes:
- Verifying against the wrong Merkle root,
- Accepting incorrect leaf formatting,
- Allowing reused proofs or claims without nonce tracking,
- Or failing to validate input boundaries (e.g., amount, address consistency).
- Attackers can exploit this to claim multiple times, forge proofs, or access restricted features.

## 🚨 Vulnerable Code

```solidity
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

contract InsecureMerkleAirdrop {
    bytes32 public merkleRoot;
    mapping(address => bool) public hasClaimed;

    function claim(bytes32[] calldata proof, uint256 amount) external {
        // ❌ Leaf is not constructed with `amount` → attacker can reuse any address-based proof
        bytes32 leaf = keccak256(abi.encodePacked(msg.sender));
        require(MerkleProof.verify(proof, merkleRoot, leaf), "Invalid proof");
        require(!hasClaimed[msg.sender], "Already claimed");

        hasClaimed[msg.sender] = true;
        // transfer tokens
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A Merkle root was generated for airdrop claims with (address, amount) as leaf nodes.
2. Contract only uses keccak256(address) for verification.
3. Attacker finds any valid proof for their address—even from a different tree or amount.
4. Contract accepts it and allows claiming without verifying correct amount.
5. In some cases, attacker can replay claim or use someone else’s proof if leaf inputs are not strict.

**Assumptions:**

- Leaf node format is inconsistent with original tree construction.
- No nonce, amount hash, or context verification.

## ✅ Fixed Code

```solidity

function claim(bytes32[] calldata proof, uint256 amount) external {
    bytes32 leaf = keccak256(abi.encodePacked(msg.sender, amount));
    require(MerkleProof.verify(proof, merkleRoot, leaf), "Invalid proof");
    require(!hasClaimed[msg.sender], "Already claimed");

    hasClaimed[msg.sender] = true;
    // transfer correct `amount` to msg.sender
}
```

## 🛡️ Prevention

### Primary Defenses

- Always hash exactly the same data structure used when generating the Merkle tree (address + amount, not just address).
- Bind claim-specific data (e.g., amount, chain ID, contract address) in the leaf hash to avoid cross-claim proof replay.

### Additional Safeguards

- Track claimed leaf hashes, not just user addresses, to prevent multiple claims with same proof.
- Include Merkle root versioning or tree rotation to expire old roots.
- Emit events upon claim verification for off-chain monitoring.

### Detection Methods

- Manual inspection of Merkle leaf construction and proof validation logic.
- Compare Merkle tree generator script with contract’s verification logic.
- Slither: custom rule to detect MerkleProof.verify() with improperly constructed leaf.

## 🕰️ Historical Exploits

- **Name:** Audius Claim Proof Bypass 
- **Date:** 2022 
- **Loss:** Users claimed multiple times by replaying proofs 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/audius-rekt/) 


## 📚 Further Reading

- [SWC-121: Incorrect Signature Verification](https://swcregistry.io/docs/SWC-121) 
- [OpenZeppelin – MerkleProof Documentation](https://docs.openzeppelin.com/contracts/4.x/api/utils#MerkleProof) 
- [Misuse of Merkle Leaf Nodes – Zokyo Auditing Tutorials](https://zokyo-auditing-tutorials.gitbook.io/zokyo-tutorials/tutorial-32-merkle-leafs/misuse-of-merkle-leaf-nodes) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Merkle Proof Misuse 
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 4  
finalScore: 4.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Unauthorized users may gain access or double-claim funds.
- **Exploitability**: Easy if leaf structure is incorrectly constructed or verified.
- **Reachability**: Affects all Merkle-based airdrops, allowlists, or claim logic.
- **Complexity**: Medium — attacker needs to understand how the leaf is hashed and what proof can be reused.
- **Detectability**: Very high with side-by-side audit of tree generation and verify() call logic.
