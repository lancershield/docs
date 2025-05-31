# Pump and Dump Tokenomics

```YAML
id: LS26H
title: Unsustainable Pump and Dump Tokenomics
baseSeverity: H
category: economic-manipulation
language: solidity
blockchain: [ethereum]
impact: User financial loss via manipulated price incentives
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.5.0", "<0.8.21"]
cwe: CWE-295
swc: SWC-136
```

## 📝 Description

- Pump and Dump Tokenomics refers to deliberately engineered or poorly designed token economics that incentivize artificial price inflation (pump), followed by a sharp decline (dump) once early actors or developers exit their positions. 
- While not always a direct smart contract bug, this economic vulnerability exploits investor psychology and trust in the system.
- In Solidity contracts, this may manifest via:
- Unfair pre-minting or high team allocations,Transfer fees that trap retail buyers,Automatic liquidity burns without accountability,Reflection mechanisms that appear rewarding but are unsustainable,Tax manipulation that can be changed post-deploy.
- Such structures are often paired with hidden backdoors or poor decentralization to enable quick developer exits.

## 🚨 Vulnerable Code

```solidity

uint256 public buyTax = 3;
uint256 public sellTax = 12;

function setTaxes(uint256 _buy, uint256 _sell) external onlyOwner {
    buyTax = _buy;
    sellTax = _sell;
}

function _transfer(address from, address to, uint256 amount) internal {
    uint256 tax = to == uniswapPair ? amount * sellTax / 100 : amount * buyTax / 100;
    uint256 netAmount = amount - tax;
    super._transfer(from, to, netAmount);
    super._transfer(from, treasury, tax);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Developers launch a token with modest buy/sell tax (e.g., 3%).
2. They accumulate liquidity and build community trust.
3. Once price peaks, sellTax is silently raised to 99%.
4. Retail holders are unable to sell while insiders dump from tax-exempt addresses.
5. Liquidity is drained, price crashes, and the project vanishes.

**Assumptions:**

- Owner can adjust tax after deployment.
- Team holds a large portion of supply or controls liquidity.
- Retail users unaware of centralized risk or lack of time lock.

## ✅ Fixed Code

```solidity

bool public taxesLocked = false;

function setTaxes(uint256 _buy, uint256 _sell) external onlyOwner {
    require(!taxesLocked, "Taxes are locked forever");
    require(_buy <= 5 && _sell <= 5, "Unreasonable tax values");
    buyTax = _buy;
    sellTax = _sell;
}

function lockTaxes() external onlyOwner {
    taxesLocked = true;
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "The protocol’s economic model causes investor loss and token collapse over time."
- context: "DeFi Yield Farm with High Emissions and No Buyback"
  severity: C
  reasoning: "Massive losses across retail users and liquidity vacuum post-hype."
- context: "Governance DAO with Controlled Emissions"
  severity: M
  reasoning: "Governance can adjust incentives and slow emissions if detected early."
```

## 🛡️ Prevention

### Primary Defenses

- Use hard-coded or capped tax rates with irreversible locking.
- Avoid owner-controlled tokenomics parameters post-launch.
- Time-lock or renounce sensitive functions (e.g., setTaxes).

### Additional Safeguards

- Vest team tokens with cliffs and linear unlocks.
- Publish a public tokenomics audit and proof-of-liquidity lock.
- Use multi-sig governance for economic parameter changes.

### Detection Methods

- Static review of tax/fee setting functions (setFee, setTax, etc.).
- Check for onlyOwner modifiers on tokenomic-critical paths.
- Audit token distribution at deployment and on-chain liquidity locks.

## 🕰️ Historical Exploits

- **Name:** BitConnect 
- **Date:** 2017–2018 
- **Loss:** Billions in investor funds  
- **Post-mortem:** [Link to post-mortem](https://techpadi.africa/2024/11/cryptos-dark-side-pumps-dumps-and-rug-pulls/?utm_source=chatgpt.com) 
- **Name:** Squid Game Token 
- **Date:** 2021 
- **Loss:** Millions in investor funds 
- **Post-mortem:** [Link to post-mortem](https://plasbit.com/research/crypto-pump-and-dump?utm_source=chatgpt.com) 

## 📚 Further Reading

- [SWC-136: Unexpected Behavior from External Calls](https://swcregistry.io/docs/SWC-136/) 
- [Chainalysis: 2021 Crypto Scam Revenues](https://www.chainalysis.com/blog/2021-crypto-scam-revenues/)
- [Coffeezilla: Tokenomics Scams Explained](https://www.youtube.com/@Coffeezilla) 

---

## ✅ Vulnerability Report

```markdown
id: LS26H
title: Unsustainable Pump and Dump Tokenomics
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 3.85
```

---

## 📄 Justifications & Analysis

- **Impact**: Users lose funds to artificial price manipulation.
- **Exploitability**: Single transaction by owner can weaponize tax.
- **Reachability**: Most tokens expose tax/fee controls publicly.
- **Complexity**: No complex logic—only configuration risk.
- **Detectability**: Tricky if disguised via obfuscated naming or hidden behind proxies.

