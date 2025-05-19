# Token URI Hijacking

```YAML
id: TBA
title: Token URI Hijacking Enables Metadata Manipulation
severity: H
category: nft-metadata
language: solidity
blockchain: [ethereum]
impact: Trust loss, fraud, or visual misrepresentation
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.5.0", "<0.8.21"]
cwe: CWE-285
swc: SWC-131
```

## 📝 Description

- Token URI Hijacking occurs when the tokenURI() function in an NFT contract is externally modifiable, poorly protected, or depends on a mutable off-chain pointer. This enables:
- Unauthorized actors (or the contract owner) to change an NFT’s metadata after mint,
- Redirecting a token’s image, name, or attributes post-sale,
- Causing trust issues, impersonation, or even scams in NFT platforms.
- The vulnerability typically stems from:
- Lack of access control in setTokenURI,Mutable baseURI without immutability guarantees,Overridable logic in tokenURI() tied to external state.

## 🚨 Vulnerable Code

```solidity

contract NFT is ERC721 {
    mapping(uint256 => string) public tokenURIs;

    function setTokenURI(uint256 tokenId, string calldata uri) external {
        tokenURIs[tokenId] = uri;
    }

    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        return tokenURIs[tokenId];
    }
}
```

## 🧪 Exploit Scenario
Step-by-step exploit process:

1. NFT is minted with legitimate metadata.
2. Malicious actor calls setTokenURI(tokenId, "https://evil.com/hacked.json").
3. NFT platforms like OpenSea or marketplaces display malicious metadata, including adult content, scam links, or fake ownership claims.
4. Users lose trust in the collection or are misled into buying fraudulent tokens.

**Assumptions:**

- setTokenURI() is public or onlyOwner but not immutable after mint.
- Off-chain metadata is mutable or hosted on insecure servers.

## ✅ Fixed Code
```solidity

contract NFT is ERC721 {
    mapping(uint256 => string) private _tokenURIs;
    mapping(uint256 => bool) private _locked;

    function _setTokenURI(uint256 tokenId, string memory uri) internal {
        require(!_locked[tokenId], "URI locked");
        _tokenURIs[tokenId] = uri;
    }

    function lockTokenURI(uint256 tokenId) external onlyOwner {
        _locked[tokenId] = true;
    }

    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        return _tokenURIs[tokenId];
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Lock metadata permanently after mint or sale.
- Use immutable IPFS hashes or on-chain metadata generation.
- Avoid exposing setTokenURI() as public.

### Additional Safeguards

- Add a lockMetadata() function post-mint.
- Use verifiable on-chain descriptors for generative projects.
- Consider emitting PermanentURI events (used by marketplaces).

### Detection Methods

- Manual inspection of setTokenURI, baseURI, and tokenURI() exposure.
- Static analysis for externally callable mutators.
- On-chain testing to simulate metadata changes post-mint.

## 🕰️ Historical Exploits

- **Name:** Off-Chain NFT Hijacking via Centralized Infrastructure 
- **Date:** 2023 
- **Loss:** Potential compromise of NFT ownership 
- **Post-mortem:** [Link to post-mortem](https://netsec.ethz.ch/publications/papers/2023-FC-NFT.pdf)
  

## 📚 Further Reading

- [SWC-117: Signature Malleability – SWC Registry](https://swcregistry.io/docs/SWC-117/) 
- [Metadata Manipulation in NFTs – OpenSea Docs](https://docs.opensea.io/docs/metadata-standards) 
- [NFT Metadata Security Best Practices – ConsenSys](https://consensys.net/blog/nft-metadata-security-best-practices/)
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Token URI Hijacking Enables Metadata Manipulation
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2    
detectability: 3  
finalScore: 3.75
```

---

## 📄 Justifications & Analysis

- **Impact**: Can mislead buyers, impersonate brands, or trigger takedowns.
- **Exploitability**: Anyone can trigger it if setter is unprotected.
- **Reachability**: tokenURI is universally called for NFTs.
- **Complexity**: Low; only logic oversight needed.
- **Detectability**: Easily detected via function signature scan and tests.


