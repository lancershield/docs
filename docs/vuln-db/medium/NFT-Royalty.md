# NFT Royalty Enforcement Bypass 

```YAML
id: TBA
title: NFT Royalty Enforcement Bypass 
baseSeverity: M
category: economic-attack
language: solidity
blockchain: [ethereum]
impact: Circumvention of creator royalties
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: medium
versions: [">0.6.0", "<0.9.0"]
cwe: CWE-284
swc: SWC-114
```

## 📝 Description

- NFT royalty bypass vulnerabilities arise when marketplaces, protocols, or users circumvent creator royalties by using non-standard transfers such as transferFrom() or safeTransferFrom() directly between peers. 
- These bypass the logic in royalty-enforcing smart contracts or off-chain systems. As a result, the intended payment to creators (e.g., 5% royalty on resale) is silently skipped, harming the long-term monetization model of NFT projects.
- This issue is especially problematic in decentralized or unregulated secondary marketplaces that ignore EIP-2981 or don’t implement enforcement logic for royalties.

## 🚨 Vulnerable Code

```solidity
// Royalty is not enforced in contract
function transferNFT(address from, address to, uint256 tokenId) external {
    require(msg.sender == from, "Unauthorized");
    _transfer(from, to, tokenId); // no royalty logic
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Alice buys an NFT from a compliant marketplace that enforces royalties (e.g., 5% to the artist).
2. Bob wants to purchase that NFT but avoid paying royalties.
3. Alice and Bob agree off-chain, and Alice transfers the NFT to Bob directly using safeTransferFrom() or a non-compliant platform.
4. The creator receives no royalty payment, and the protocol’s royalty system is circumvented.

**Assumptions:**

- No enforcement mechanism at the smart contract level (only off-chain).
- Protocol relies on goodwill or marketplace compliance with EIP-2981.
- Peer-to-peer transfers are unrestricted.

## ✅ Fixed Code

```solidity

function royaltyEnforcedTransfer(address from, address to, uint256 tokenId) external payable {
    uint256 royalty = calculateRoyalty(tokenId);
    require(msg.value >= royalty, "Royalty not paid");
    payable(royaltyReceiver).transfer(royalty);

    _transfer(from, to, tokenId);
}
```
## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: M
  reasoning: "Moderate financial impact for most NFT collections; depends on royalty rates and trading volume."
- context: "High-profile NFT collection with royalty-dependent revenue"
  severity: H
  reasoning: "Severe impact on creator income and ecosystem incentives."
- context: "On-chain royalty-enforced NFT contract"
  severity: L
  reasoning: "Mitigated by smart contract-enforced mechanisms."
```

## 🛡️ Prevention

### Primary Defenses

- Implement royalty-enforced transfer functions on-chain.
- Use onlyRoyaltyCompliantTransfer() and disable open transferFrom() unless needed.
- Lock down transfers to marketplaces that implement royalty enforcement (e.g., OpenSea Operator Filter Registry).

### Additional Safeguards

- Use EIP-2981 interface in all NFT contracts and register with compliant marketplaces.
- Leverage Operator Registry filters to block royalty-circumventing contracts.
- Use a wrapper contract to enforce royalties on resale.

### Detection Methods

- Static analysis for presence of EIP-2981 interface.
- Check for unrestricted transferFrom() or safeTransferFrom() usage.
- Manual review of transfer hooks (e.g., _beforeTokenTransfer, onERC721Received).

## 🕰️ Historical Exploits

- **Name:** OpenSea Pro v3 Royalty Enforcement Bypass 
- **Date:** April 2025 
- **Loss:** Undisclosed losses due to royalty circumvention
- **Post-mortem:** [Link to post-mortem](https://markaicode.com/opensea-pro-v3-royalty-enforcement-bypass-protection/)

## 📚 Further Reading

- [EIP-2981: NFT Royalty Standard](https://eips.ethereum.org/EIPS/eip-2981) 
- [Why NFT royalties are almost impossible to enforce on-chain – The Block](https://www.theblock.co/post/178603/why-nft-royalties-are-almost-impossible-to-enforce-on-chain) 
- [OpenSea’s royalty enforcement tool Operator Filter to be turned off – Cryptopolitan](https://www.cryptopolitan.com/openseas-operator-filter-to-be-turned-off/) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: NFT Royalty Bypass 
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Royalties — core to creator economy — are nullified.
- **Exploitability**: Easily done through any non-compliant contract or interface.
- **Reachability**: Applies broadly to all NFTs using ERC-721 or ERC-1155.
- **Complexity**: Moderate, as it involves just choosing the wrong platform.
- **Detectability**: Hard to catch unless transfers are explicitly restricted or monitored.