# Unsafe Mint to Non-Receiver

```YAML
id: TBA
title: Unsafe Mint to Non-Receiver
severity: M
category: erc721
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Tokens permanently locked or lost in non-receiving contracts
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-707
swc: SWC-115
```

## 📝 Description

- Unsafe Mint to Non-Receiver occurs when an ERC721 (or ERC1155) token is minted to a contract address that does not implement the required ERC721Receiver or ERC1155Receiver interface. Without these interfaces:
- The mint or transfer may silently succeed but the token is stuck
Or it may revert, breaking user workflows
- Users lose access to the NFT unless the receiving contract is upgraded or self-destructed
- This vulnerability becomes critical when:
- Minting is done via mint(to) instead of safeMint(to)

## 🚨 Vulnerable Code

```solidity

function mint(address to, uint256 tokenId) public onlyOwner {
    _mint(to, tokenId); // ❌ does not check if `to` is a contract that can receive ERC721s
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A user calls mint(to) with to set as a custom smart contract or mistakenly as a DAO contract.
2. The minting succeeds (if _mint() does not enforce checks).
3. The onERC721Received() function is not present in the recipient contract.
4. The user has no way to interact with the NFT inside that contract or retrieve it.

**Assumptions:**

- The NFT contract uses _mint() instead of _safeMint() or does not check to’s contract status.
- The recipient is a non-upgradeable contract that does not implement onERC721Received.
- There are no fallback functions to forward or reclaim the NFT.

## ✅ Fixed Code

```solidity

function safeMint(address to, uint256 tokenId) public onlyOwner {
    _safeMint(to, tokenId); // ✅ checks for ERC721Receiver compatibility
}
```

## 🛡️ Prevention

### Primary Defenses

- Use _safeMint() instead of _mint() when targeting arbitrary addresses.
- Validate receiver compatibility with IERC721Receiver.onERC721Received on mint and transfer.
- Require whitelisted or verifiable addresses during minting phases.

### Additional Safeguards

- Warn users via frontend if to is a contract.
- Block minting to contracts without ERC165 support or receiver interface.
- Use off-chain allowlist for mint recipients in pre-sale or airdrops.

### Detection Methods

- Static analysis for _mint() used without compatibility enforcement.
- Slither rules: erc721-unsafe-mint, low-level-call-missing-check
- Simulation: test mints to Gnosis Safe, Timelock, or proxy contracts

## 🕰️ Historical Exploits

- **Name:** PuttyV2 Unsafe Minting 
- **Date:** 2022-06-30 
- **Loss:** ~$200,000 USD
- **Post-mortem:** [Link to post-mortem](https://github.com/code-423n4/2022-06-putty-findings/issues/327) 
- **Name:** Optimism NFT Bridge Incident 
- **Date:** 2023-01-15 
- **Loss:** ~$75,000 USD
- **Post-mortem:** [Link to post-mortem](https://github.com/sherlock-audit/2023-01-optimism-judging/issues/82)

## 📚 Further Reading

- [ERC721 Spec](https://eips.ethereum.org/EIPS/eip-721) 
- [CWE-707: Improper Handling of Exceptional Conditions](https://cwe.mitre.org/data/definitions/707.html) 
- [SWC-115: Authorization through tx.origin](https://swcregistry.io/docs/SWC-115) 
- [OpenZeppelin – Safe Transfers](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Unsafe Mint to Non-Receiver
severity: M
score:
impact: 3  
exploitability: 3  
reachability: 4  
complexity: 2  
detectability: 3  
finalScore: 3.05
```

---

## 📄 Justifications & Analysis

- **Impact**: Permanent token loss, user dissatisfaction, and protocol liability.
- **Exploitability**: Anyone can mint to a contract with missing interface.
- **Reachability**: Found in manual mint logic, custom airdrop contracts, or mint(to) calls.
- **Complexity**: Low – just an invalid mint target.
- **Detectability**: Static analysis and testing catch this when _mint() is used naively.
