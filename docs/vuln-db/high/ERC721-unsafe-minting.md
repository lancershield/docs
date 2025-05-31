# ERC721 Unsafe Minting 

```YAML
id: LS35H
title: ERC721 Unsafe Minting 
baseSeverity: H
category: minting
language: solidity
blockchain: [ethereum]
impact: Unauthorized token creation, token loss, or metadata corruption
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-284
swc: SWC-122
```

## 📝 Description

- In ERC721 implementations, unsafe or unchecked minting logic can lead to:
- Duplicate token IDs being minted to multiple users
- Minting to the zero address, which locks tokens forever
- Bypassing hooks like ERC721Receiver.onERC721Received() when minting to contracts
- Incorrect ownership tracking, which breaks downstream transfers or approvals
- This is especially dangerous in manually written _mint() functions that don’t adhere to the ERC-721 specification, or when using outdated or partial implementations that skip validation steps.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract BrokenERC721 {
    mapping(uint256 => address) public ownerOf;

    function mint(uint256 tokenId, address to) external {
        ownerOf[tokenId] = to; // ❌ no zero address check, no uniqueness check
    }
}
```

## 🧪 Exploit Scenario

1. A malicious user calls mint(123, 0x000...0000), locking token 123 permanently.
2. Another user later tries to mint 123 and is mistakenly shown as owner, though transfers break.
3. A frontend marketplace shows incorrect ownership or fails to list due to invalid state.
4. If the token is minted to a contract (e.g., auction, wrapper), the lack of onERC721Received() call breaks receiver logic or causes silent token loss.

**Assumptions:**

- Custom minting logic is implemented without safeguards.
- Contract is accessible by external users or minters.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeERC721 is ERC721, Ownable {
    constructor() ERC721("SafeToken", "SAFE") {}

    function safeMint(address to, uint256 tokenId) external onlyOwner {
        require(to != address(0), "Cannot mint to zero address");
        require(!_exists(tokenId), "Token already exists");

        _safeMint(to, tokenId); // ✅ calls onERC721Received for contracts
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Minting NFTs to unknown or user-supplied addresses"
  severity: H
  reasoning: "Tokens can be permanently locked and lost, affecting protocol credibility"
- context: "Minting only to trusted EOAs or verified contract addresses"
  severity: L
  reasoning: "Risk is mitigated if all recipients are known to be compatible"
- context: "Use of safeMint with receiver validation"
  severity: I
  reasoning: "Implementation is secure and in line with ERC721 standards"
```

## 🛡️ Prevention

### Primary Defenses

- Always use _safeMint() unless you know the receiver is an EOA.
- Check for zero address and duplicate token IDs before minting.

### Additional Safeguards

- Restrict minting to trusted actors via onlyOwner, onlyMinter, or allowlists.
- Emit standard Transfer(address(0), to, tokenId) events during minting.

### Detection Methods

- Audit all mint() implementations for:
- address(0) checks
- exists(tokenId) validation
- Use of _safeMint() vs _mint()
- Tools: Slither (missing-zero-check, duplicate-token-id), MythX

## 🕰️ Historical Exploits

- **Name:** HypeBears NFT Reentrancy Exploit 
- **Date:** 2022-02 
- **Loss:** $150,000
- **Post-mortem:** [Link to post-mortem](https://blocksecteam.medium.com/when-safemint-becomes-unsafe-lessons-from-the-hypebears-security-incident-2965209bda2a)
  
## 📚 Further Reading

- [ERC-721 Specification](https://eips.ethereum.org/EIPS/eip-721)
- [OpenZeppelin ERC721 Docs](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721)
- [SWC-122: Lack of Proper Validation](https://swcregistry.io/docs/SWC-122/) 
- [Solidity Docs – safeMint vs mint](https://docs.soliditylang.org/) 

---

## ✅ Vulnerability Report

```markdown
id: LS35H
title: ERC721 Unsafe Minting 
severity: H
score:
impact: 4   
exploitability: 3 
reachability: 4  
complexity: 2  
detectability: 5 
finalScore: 3.65
```

---

## 📄 Justifications & Analysis

- **Impact**: Leads to permanent token loss, broken ownership, or unlistable NFTs.
- **Exploitability**: Any user can mint if access control is weak or missing checks.
- **Reachability**: Very common in NFT projects, especially forks and hand-written contracts.
- **Complexity**: Conceptually simple but often overlooked.
- **Detectability**: High—can be caught with basic review and tooling.