# Minting to Burn Address 

```YAML
id: LS06M
title: Minting to Burn Address 
baseSeverity: M
category: tokenomics
language: solidity
blockchain: [ethereum]
impact: Tokens minted are irrecoverable, reducing total usable supply and confusing metrics
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-707
swc: SWC-136
```

## 📝 Description

- Minting to the burn address (commonly `0x0000000000000000000000000000000000000000` or `0x000...dEaD`) means tokens are sent to an address with no private key or recovery mechanism, resulting in:
- Permanent removal from circulation
- Inaccurate supply metrics (e.g., total supply != sum of balances),
- Confusion for indexers, explorers, and dApps relying on full token accounting.
- This may occur due to misconfigured rewards, staking, or airdrop logic that defaults to a zero or burn address instead of a valid recipient.

## 🚨 Vulnerable Code

```solidity
function mintTo(address recipient, uint256 amount) external onlyMinter {
    _mint(recipient, amount); // ❌ recipient could be 0x0 or 0xdead
}
```

## 🧪 Exploit Scenario

Step-by-step impact:

1. A misconfigured airdrop attempts to mint tokens to address(0) due to uninitialized recipient.
2. Tokens are successfully minted to the zero address using _mint(...).
3. These tokens are now permanently unusable and unrecoverable.
4. Protocol reports total supply higher than circulating supply, causing accounting errors.

**Assumptions:**

- No validation on recipient address before minting.
- Minter logic is callable via governance, owner, or automated scripts.

## ✅ Fixed Code

```solidity
function mintTo(address recipient, uint256 amount) external onlyMinter {
    require(recipient != address(0), "Cannot mint to zero address");
    require(recipient != 0x000000000000000000000000000000000000dEaD, "Cannot mint to burn address");

    _mint(recipient, amount); // ✅ Safe mint
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Tokens are unrecoverable but damage is limited to accounting inconsistencies."
- context: "DeFi protocol with dynamic supply"
  severity: H
  reasoning: "Minting errors can distort LP ratios, governance weights, or vault strategies."
- context: "Governance token with voting power from totalSupply"
  severity: C
  reasoning: "Burned tokens may tilt decisions or cause vote inflation."
```

## 🛡️ Prevention

### Primary Defenses

- Always validate recipient is neither address(0) nor known burn addresses (0x...dead).
- Define an internal _isValidRecipient() check for all minting logic.
- Include such validation in both mint and airdrop functions.

### Additional Safeguards

- Add tests for invalid recipient edge cases.
- Include metrics on how much supply is burned vs circulating.
- For intentional burns, use transfer(...) rather than mint(...).

### Detection Methods

- Slither: mint-to-zero-address, unsafe-recipient, irrecoverable-mint detectors.
- Manual audit of all _mint(...) and mintTo(...) usages.
- Test with empty or unset recipient values to simulate misconfiguration.

## 🕰️ Historical Incidents

- **Name:** Paid Network Exploit 
- **Date:** 2021 
- **Loss:** Approximately $180 million 
- **Post-mortem:** [Link to post-mortem](https://heraldsheets.com/infinite-mint-attacks-all-you-need-to-know/)

## 📚 Further Reading

- [SWC-136: Unexpected Behavior (Burn Address Issues)](https://swcregistry.io/docs/SWC-136) 
- [OpenZeppelin ERC20 _mint() Docs](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-) 
- [Token Standards and Burn Semantics](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/) 

--- 

## ✅ Vulnerability Report 

```markdown
id: LS06M
title: Minting to Burn Address 
severity: M
score:
impact: 3         
exploitability: 0 
reachability: 5   
complexity: 1    
detectability: 5  
finalScore: 2.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Medium — permanently locks supply and can distort token economics.
- **Exploitability**: None by an attacker, but dangerous as a self-inflicted bug.
- **Reachability**: Common in airdrop, reward, and staking contracts.
- **Complexity**: Very low — usually caused by missing address validation.
- **Detectability**: High — both tools and manual review catch this.