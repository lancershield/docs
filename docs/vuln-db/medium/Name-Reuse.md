# Contract Name Reuse Confusion

```YAML
id: LS12M
title: Contract Name Reuse Confusion
baseSeverity: M
category: naming
language: solidity
blockchain: [ethereum]
impact: User deception, phishing risk, or fund misdirection
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-706
swc: SWC-131
```

## 📝 Description

- Reusing the same name() or symbol() value as a well-known token or contract can cause end-user confusion, misrouted transactions, or phishing attacks. 
- This occurs when malicious or negligent developers deploy a new token or proxy contract using a name identical to a trusted brand (e.g., "USDT", "UNI", "DAI").
- Since name() and symbol() are not enforced to be unique on-chain, wallets, explorers, and dApps may display or auto-select the wrong token. 
- When paired with other vulnerabilities (e.g., transfer() abuse, low liquidity slippage), this can lead to user losses or scam propagation.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract FakeDAI is ERC20 {
    constructor() ERC20("DAI Stablecoin", "DAI") {
        _mint(msg.sender, 1_000_000 * 10**18);
    }
}
```
## 🧪 Exploit Scenario

1. An attacker deploys a new ERC-20 contract using the name "USDT" and symbol "USDT".
2. They create a liquidity pool on a DEX (e.g., Uniswap, PancakeSwap).
3. Unsuspecting users add the token manually or interact via copy-pasted links.
4. Wallets display "USDT", and users believe it's legitimate.
5. The attacker front-runs or drains user funds by manipulating liquidity or permissions.

**Assumptions:**

- Token name/symbol matches a widely-used project.
- Users rely on name/symbol instead of address validation.
- Wallets and UIs do not warn about duplicate names.

## ✅ Fixed Code

```solidity
// Fixed version with unique contract names
contract ERC20TokenA {
    string public name = "TokenA";
}

contract ERC20TokenB {
    string public name = "TokenB";
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "No direct exploit path, but misleading behavior can confuse integrators."
- context: "Protocol mimicking major token or router names"
  severity: H
  reasoning: "High potential for phishing, fraud, or UX deception."
- context: "Internal-use contracts with unique names"
  severity: L
  reasoning: "Minimal confusion if naming is consistent and internal."
```

## 🛡️ Prevention

### Primary Defenses

- Always verify contract addresses, not just token names.
- Wallets and dApps should display token metadata (source, creator, address).
- Reputable projects should register tokens with ENS, GitHub, or trusted registries.

### Additional Safeguards

- Use on-chain registries like EIP-1820 for interface resolution.
- Apply domain-based ownership to contract naming (e.g., via ENS reverse lookup).
- Educate users to double-check token addresses before transacting.

### Detection Methods

- Scan for token contracts using name()/symbol() identical to known brands.
- Check if a token is missing additional metadata or verified source code.
- Tools: TokenLists Validator, Etherscan labels, block explorer APIs

## 🕰️ Historical Exploits

- **Name:** Uniswap Fake Token Phishing Scam
- **Date:** July 2022 
- **Loss:** Over $4.7 million 
- **Post-mortem:** [Link to post-mortem](https://cointelegraph.com/news/more-than-4-7m-stolen-in-uniswap-fake-token-phishing-attack)
  
## 📚 Further Reading

- [SWC-131: Clone Functionality with Misleading Naming](https://swcregistry.io/docs/SWC-131/) 
- [Understanding Token Impersonation Attacks – ConsenSys](https://consensys.net/blog/security/understanding-token-impersonation-attacks/) 
- [Best Practices for Token Naming – OpenZeppelin](https://docs.openzeppelin.com/contracts/4.x/erc20) 

---

## ✅ Vulnerability Report

```markdown
id: LS12M
title: Contract Name Reuse Confusion
severity: M
score:
impact: 3         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 3.4
```

---

## 📄 Justifications & Analysis

- **Impact**: While the vulnerability doesn’t alter on-chain state, it leads to real economic loss via deception or user error.
- **Exploitability**: Anyone can deploy a clone token with the same name and list it on DEXs.
- **Reachability**: Affects wallet interfaces, explorers, and unsuspecting users across the ecosystem.
- **Complexity**: Very easy to execute with basic ERC-20 code and liquidity.
- **Detectability**: Requires off-chain curation or metadata validation; static tools may miss the social engineering aspect.

