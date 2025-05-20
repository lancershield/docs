# Incorrect ERC-721 Interface Causes NFT Transfer Failures

```YAML
id: TBA
title: Incorrect ERC-721 Interface Causes NFT Transfer Failures or Loss of Ownership
severity: H
category: interface-mismatch
language: solidity
blockchain: [ethereum]
impact: Failed transfers, stuck NFTs, or unauthorized approvals
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-704
swc: SWC-135
```
📝 Description

- The ERC-721 NFT standard defines strict interfaces for token transfer and ownership operations, such as safeTransferFrom, transferFrom, and approve. 
- A mismatch between the interface declared in consuming contracts and the actual implementation in deployed token contracts can lead to:
- Reverts when calling non-existent or improperly defined functions
- NFTs being stuck in contracts due to missing receiver handling
- Contracts assuming ownership changes without validation
- This vulnerability often occurs when:
- IERC721 is incorrectly defined (e.g., omitting returns values)
- Contracts interact with non-standard or minimalistic NFT contracts
- The function selector does not match due to interface misdefinition

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IBrokenERC721 {
    function transferFrom(address from, address to, uint256 tokenId) external; // ❌ Missing returns(bool)
}

contract NFTVault {
    function deposit(IBrokenERC721 token, uint256 tokenId) external {
        token.transferFrom(msg.sender, address(this), tokenId); // ❌ Will revert if token expects safeTransferFrom or returns(bool)
    }
}
```

## 🧪 Exploit Scenario

1. A vault accepts NFTs by calling transferFrom via a custom IERC721 interface.
2. An NFT is passed in with a slightly different function signature (e.g., custom transferFrom with return value).
3. The call reverts due to incorrect function selector.
4. User loses gas and cannot deposit the NFT—or worse, the NFT is transferred but logic assumes otherwise, leading to stuck or irrecoverable tokens.

**Assumptions:**

- Consuming contracts do not use the correct OpenZeppelin IERC721 interface.
- NFT contracts do not follow full compliance with ERC-721 metadata or safe transfer requirements.

## ✅ Fixed Code

```solidity
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

contract NFTVault {
    function deposit(IERC721 token, uint256 tokenId) external {
        token.transferFrom(msg.sender, address(this), tokenId); // ✅ Uses standardized interface
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use canonical OpenZeppelin IERC721 interface.
- Avoid custom ERC-721 interfaces unless fully matched to deployed token ABI.

### Additional Safeguards

- Check return values (where applicable).
- Implement and register onERC721Received when receiving NFTs to contracts.

### Detection Methods

- Audit interfaces for selector mismatches or missing function signatures.
- Compare custom interfaces against OpenZeppelin ERC-721 reference.
- Tools: Slither (incorrect-interface), Hardhat typechain, manual ABI inspection

## 🕰️ Historical Exploits

- **Name:** NFTX Vault NFT Lock Bug 
- **Date:** 2021 
- **Loss:** NFTs stuck in vault due to interface mismatch
- **Post-mortem:** [Link to post-mortem](https://discord.gg/nftx) 
  
## 📚 Further Reading

- [SWC-135: Incorrect Authorization](https://swcregistry.io/docs/SWC-135/) 
- [ERC-721 Standard Spec](https://eips.ethereum.org/EIPS/eip-721)
- [OpenZeppelin IERC721 Interface](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/IERC721.sol) 
- [Solidity Function Selector Mismatch Explained](https://blog.soliditylang.org/2021/04/21/custom-errors/) 

## ✅ Vulnerability Report
```markdown
id: TBA
title: Incorrect ERC-721 Interface Causes NFT Transfer Failures or Loss of Ownership
severity: H
score:
impact: 4 
exploitability: 3 
reachability: 3   
complexity: 2    
detectability: 4  
finalScore: 3.45
```

---

## 📄 Justifications & Analysis

- **Impact**: Misbehavior or failure of NFT-related operations can cause stuck assets or unauthorized access.
- **Exploitability**: Attackers can exploit contracts assuming wrong ownership or execution logic.
- **Reachability**: Any function interacting with ERC-721 tokens is at risk.
- **Complexity**: The mistake is simple—interface inconsistency, but subtle.
- **Detectability**: Missed without strong typing or ABI-based contract testing.