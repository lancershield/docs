# Broken Interface Compatibility

```YAML
id: TBA
title: Broken Interface Compatibility 
severity: M
category: interface-mismatch
language: solidity
blockchain: [ethereum]
impact: Contracts may fail to interact with external systems or violate protocol assumptions
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-706
swc: SWC-111

```

## 📝 Description

- Broken interface compatibility occurs when a contract claims to implement a standard interface (e.g., ERC20, ERC721, ERC1155), but:
- Fails to match the expected function signatures,Omits required return values or events,Deviates from expected revert/return behavior.
- This results in:Silent failures in dApps, DeFi protocols, or bridges,Inability for wallets and explorers to index or interact with tokens.
- Security bypasses in access-control or transfer logic that rely on interface detection.

## 🚨 Vulnerable Code

```solidity
// ❌ Incorrect ERC20 transfer (missing return value)
function transfer(address to, uint256 amount) public {
    _transfer(msg.sender, to, amount); // Does not return (bool)
}
```


## 🧪 Exploit Scenario

Step-by-step failures:

1. A token contract claims to be ERC20-compliant, but transfer() does not return bool.
2. A DeFi vault tries to call transfer(...) and expects true, but receives undefined behavior.
3. The transaction fails silently or executes incorrectly, potentially causing loss of funds or broken integrations.

**Assumptions:**

- An NFT marketplace calls safeTransferFrom() on an ERC721 token.
- The token does not validate onERC721Received, causing tokens to get locked in the contract.

## ✅ Fixed Code

```solidity

// ✅ ERC20-compatible transfer
function transfer(address to, uint256 amount) public returns (bool) {
    _transfer(msg.sender, to, amount);
    return true;
}

// ✅ ERC721-compatible safe transfer
function safeTransferFrom(address from, address to, uint256 tokenId) public override {
    _transfer(from, to, tokenId);
    require(_checkOnERC721Received(from, to, tokenId), "ERC721: transfer to non-ERC721Receiver");
}

```
## 🛡️ Prevention

### Primary Defenses

- Always inherit from official OpenZeppelin interfaces (e.g., IERC20, IERC721, IERC1155).
- Use automated interface conformance tools such as supportsInterface(bytes4).

### Additional Safeguards

- Include tests for external integrations, such as calls from Uniswap, OpenSea, or Etherscan.
- Ensure all interface-required return values (bool, bytes4, etc.) are respected.
- Avoid manually rewriting standard functions—prefer using battle-tested libraries.

### Detection Methods

- Slither: incorrect-interface, missing-return, incomplete-erc20/erc721 detectors.
- Hardhat or Foundry tests using expectCall() to simulate DeFi usage.
- supportsInterface() validation via ERC165 for all interface claims.

## 🕰️ Historical Incidents

- **Name:** USDT (Tether) ERC20 Compatibility Issue 
- **Date:** 2017–present 
- **Impact:** Tether does not return `bool` in `transfer()`, breaking some DeFi logic 
- **Post-mortem:** [Link](https://github.com/ethereum/EIPs/issues/20) 

## 📚 Further Reading

- [SWC-111: ERC20 Compliance Violation](https://swcregistry.io/docs/SWC-111) 
- [OpenZeppelin ERC Standards](https://docs.openzeppelin.com/contracts/) 
- [ERC20, ERC721, ERC1155 Specs](https://eips.ethereum.org/) 
- [Solidity Interface Design](https://docs.soliditylang.org/en/latest/contracts.html#interfaces)


---

## ✅ Vulnerability Report


```markdown

id: TBA
title: Broken Interface Compatibility 
severity: M
score:
impact: 4         
exploitability: 2 
reachability: 5   
complexity: 2     
detectability: 5  
finalScore: 3.6


```

---

## 📄 Justifications & Analysis

- **Impact**: High — downstream protocols and UIs may fail silently or misbehave.
- **Exploitability**: Low-to-medium — typically results in DoS, not theft.
- **Reachability**: Very common in custom-written ERC20/ERC721 logic.
- **Complexity**: Easy to fix by inheriting from OpenZeppelin standards.
- **Detectability**: High — static analyzers and audit checklists catch this fast.