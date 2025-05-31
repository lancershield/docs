# Mutable On-Chain Metadata

```YAML
id: LS29M
title: Mutable On-Chain Metadata
baseSeverity: M
category: metadata
language: solidity
blockchain: [ethereum, polygon, arbitrum, optimism, bsc]
impact: Trust erosion, rugpull risk, and incorrect display of NFTs
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-494: Download of Code Without Integrity Check
swc: SWC-138: Unencrypted Sensitive Data on Chain
```

## 📝 Description

- Mutable On-Chain Metadata occurs when an NFT or token contract allows the modification of metadata URIs or token descriptors after mint, typically through functions like setBaseURI, setTokenURI, or full rewrites to token metadata mappings.
- This introduces serious trust and security risks:
- Creators or admins can swap images, traits, or even remove metadata
- Marketplaces display incorrect or misleading asset visuals
- Tokens become untrustworthy over time (e.g., "rugpull" NFTs)
- This design violates the immutability principle of NFTs and undermines interoperability across dApps, games, and marketplaces.

## 🚨 Vulnerable Code

```solidity

string private baseURI;

function setBaseURI(string calldata _uri) external onlyOwner {
    baseURI = _uri; // ❌ mutable metadata source
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. An NFT project mints thousands of tokens with attractive art.
2. baseURI is mutable and stored on a centralized or editable server (e.g., IPFS with modifiable gateway or HTTP server).
3. After users purchase the NFTs, the project owner calls setBaseURI(), updating metadata to:

**Assumptions:**

- The contract owner or admin retains control over metadata pointers (e.g., baseURI, tokenURI).
- There is no immutable, finalized, or locked flag post-mint.
- NFT data (art, traits, metadata.json) is hosted off-chain or through a mutable pointer.

## ✅ Fixed Code

```solidity

string private baseURI;
bool public metadataFrozen;

function setBaseURI(string calldata _uri) external onlyOwner {
    require(!metadataFrozen, "Metadata is frozen");
    baseURI = _uri;
}

function freezeMetadata() external onlyOwner {
    metadataFrozen = true;
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Trust is affected, but no direct asset loss."
- context: "NFT project with post-sale metadata updates"
  severity: H
  reasoning: "Metadata tampering leads to reputational damage and user loss."
- context: "Verified immutable metadata with freeze option"
  severity: L
  reasoning: "Only legacy misuse cases remain due to freezing mechanism."
```

## 🛡️ Prevention

### Primary Defenses

- Use freezeMetadata() or permanent URIs after mint.
- Prefer fully on-chain metadata or IPFS CID pinned content.
- Emit MetadataFrozen() event for marketplace and user verification.

### Additional Safeguards

- Store hash of metadata on-chain to verify integrity.
- Implement permissionless access to freeze metadata after a deadline.
- Use decentralized pinning services or Arweave for immutability.

### Detection Methods

- Static scan for setBaseURI, setTokenURI, or modifiable metadata variables.
- Check deployer privileges and access control around URI modification.
- Tools: Slither (unprotected-write), OpenSea's metadata freeze scanner

## 🕰️ Historical Exploits

- **Name:** Pixelmon NFT Reveal Controversy 
- **Date:** 2022-02 
- **Loss:** ~$70,000,000 (raised) 
- **Post-mortem:** [Link to post-mortem](https://decrypt.co/92653/pixelmon-ethereum-nft-metaverse-game-70m) 

## 📚 Further Reading

- [EIP-721: Non-Fungible Token Standard](https://eips.ethereum.org/EIPS/eip-721)
- [OpenZeppelin – Metadata URI Freezing](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721URIStorage) 
- [OpenSea Metadata Standards](https://docs.opensea.io/docs/metadata-standards)
- [Slither Static Analyzer](https://github.com/crytic/slither)

---

## ✅ Vulnerability Report

```markdown
id: LS29M
title: Mutable On-Chain Metadata
severity: M
score:
impact: 3  
exploitability: 3  
reachability: 4  
complexity: 2  
detectability: 3  
finalScore: 3.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Can irreversibly destroy NFT value and damage marketplace trust.
- **Exploitability**: Easily triggered by contract owner without user consent.
- **Reachability**: Common in early NFT contracts and poorly-audited mints.
- **Complexity**: Very low – calling a setter function.
- **Detectability**: Detectable via Slither and audit of URI-related logic.
