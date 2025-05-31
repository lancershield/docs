# NFT Metadata Tampering

```YAML
id: LS22H
title: NFT Metadata Tampering 
baseSeverity: H
category: metadata-integrity
language: solidity
blockchain: [ethereum]
impact: NFT holders or marketplaces may be deceived by altered or manipulated metadata
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-345
swc: SWC-131
```

## 📝 Description

- NFT metadata tampering occurs when the token URI (`tokenURI`) or metadata contents of an NFT can be changed after minting, allowing the project owner or a malicious actor to:
- Switch traits or visuals, modifying rarity after reveal,
- Inject harmful or fraudulent content,
- Mislead collectors or marketplaces about the true identity or utility of the token.
- This breaks the core principle of immutability and trustlessness, particularly in projects that use centralized servers or lack URI locking mechanisms.

## 🚨 Vulnerable Code

```solidity
contract MutableNFT is ERC721 {
    mapping(uint256 => string) private _tokenURIs;

    function setTokenURI(uint256 tokenId, string memory uri) external onlyOwner {
        _tokenURIs[tokenId] = uri; // ❌ Mutable after mint
    }

    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        return _tokenURIs[tokenId];
    }
}
```

## 🧪 Exploit Scenario

Step-by-step manipulation:

1. Project deploys an NFT collection where tokenURI can be changed post-mint.
2. A buyer purchases a rare NFT with a valuable trait (e.g., “gold background”).
3. After the sale, the project owner modifies the URI, removing the rare trait or replacing it with a common one.
4. The holder now owns a different NFT than what was advertised.

**Assumptions:**

- An attacker gains admin access to a metadata server and changes JSON files without on-chain auditability.
- Marketplaces and collectors rely on incorrect metadata during sale or valuation.

## ✅ Fixed Code

```solidity

contract ImmutableNFT is ERC721 {
    mapping(uint256 => string) private _tokenURIs;
    bool public metadataFrozen;

    function setTokenURI(uint256 tokenId, string memory uri) external onlyOwner {
        require(!metadataFrozen, "Metadata frozen");
        _tokenURIs[tokenId] = uri;
    }

    function freezeMetadata() external onlyOwner {
        metadataFrozen = true; // ✅ Locks all metadata forever
    }

    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        return _tokenURIs[tokenId];
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Public NFT project with mutable metadata after mint"
  severity: H
  reasoning: "Severe user deception and financial harm possible"
- context: "Private or artistic project with disclosed mutability"
  severity: M
  reasoning: "Impact depends on transparency and buyer expectations"
- context: "NFT metadata and images fully content-addressed and immutable"
  severity: I
  reasoning: "Vulnerability mitigated by design"
```

## 🛡️ Prevention

### Primary Defenses

- Lock metadata permanently using freezeMetadata() or by removing setters post-reveal.
- Host metadata on decentralized, immutable storage like IPFS or Arweave.
- Emit events when metadata is revealed or frozen (MetadataFrozen()).

### Additional Safeguards

- Make tokenURI deterministic and based on token ID (e.g., baseURI + id).
- Use access controls and audit trails for any metadata editing tools.
- Store content hashes on-chain to verify external metadata integrity.

### Detection Methods

- Slither: mutable-uri, post-mint-uri-change, admin-controlled-uri detectors.
- Manual audits for any setTokenURI, updateURI, or mutable metadata patterns.
- Check for dynamic tokenURI() logic relying on off-chain variables.

## 🕰️ Historical Exploits

- **Name:** Bored Ape Yacht Club Metadata Tampering Incident 
- **Date:** 2021 
- **Loss:** Potential reputational damage and user trust erosion
- **Post-mortem:** [Link to post-mortem](https://medium.com/%40NoamaSamreen/nft-security-6d3d8063c834) 


## 📚 Further Reading

- [SWC-131: Incorrect Calculation or Relying on External Values](https://swcregistry.io/docs/SWC-131) 
- [OpenZeppelin ERC721 Metadata Guidelines](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721) 
- [Best Practices for NFT Metadata](https://docs.opensea.io/docs/metadata-standards) 

---

## ✅ Vulnerability Report 

```markdown
id: LS22H
title: NFT Metadata Tampering 
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 3.85
```

---

## 📄 Justifications & Analysis

- **Impact**: High — alters perceived value and breaks trust in NFT immutability.
- **Exploitability**: Medium — requires access, but often retained by project owners.
- **Reachability**: Widespread in early-stage or lazy-mint projects.
- **Complexity**: Low — one mutable mapping or setter function is all it takes.
- **Detectability**: High — clear in audit and tooling scans.