# Inconsistent Decimal Precision 

```YAML

id: TBA
title: Inconsistent Decimal Precision 
severity: M
category: precision
language: solidity
blockchain: [ethereum]
impact: Incorrect token amounts, failed integrations, or mispriced operations
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<latest"]
cwe: CWE-1284
swc: SWC-136

```

## 📝 Description

- Inconsistent decimal precision arises when a contract assumes all tokens use a fixed number of decimals (commonly 18), while interacting with tokens that use different decimal configurations (e.g., 6, 8, or 0). 
- If these differences are not accounted for:
- Math operations produce incorrect results,
- Token transfers, rewards, and price calculations break,
- On-chain accounting and oracle readings may become invalid.
- This issue is common in:
- DEX routers and aggregators,
- Vault and reward calculations,
- Token conversion logic (e.g., stablecoins vs. native tokens).

## 🚨 Vulnerable Code

```solidity
function deposit(address token, uint256 amount) external {
    require(amount > 0, "Invalid amount");

    // ❌ Assumes token uses 18 decimals
    uint256 normalized = amount * 1e18;
    // Logic continues assuming 18 decimals...
}
```

## 🧪 Exploit Scenario

Step-by-step failure:

1. A user deposits 1 USDC (6 decimals) into a vault expecting to receive 1 unit of value.
2. Contract multiplies by 1e18 without accounting for decimals() == 6.
3. Logic misinterprets 1 USDC as 1e24 units.
4. Vault reward logic breaks, either over-rewarding or under-crediting the user.

**Assumptions:**

- The contract does not fetch or normalize using IERC20(token).decimals().
- Different tokens with varying decimals interact through a shared logic path.

## ✅ Fixed Code

```solidity

function normalizeAmount(address token, uint256 amount) public view returns (uint256) {
    uint8 decimals = IERC20Metadata(token).decimals(); // ✅ dynamically fetch decimals
    return amount * (10 ** (18 - decimals)); // Convert to 18-decimal standard
}

```
Or store token-specific decimal normalization factors during configuration:

```solidity

mapping(address => uint8) public tokenDecimals;

function setTokenDecimals(address token, uint8 decimals) external onlyOwner {
    tokenDecimals[token] = decimals;
}

```


## 🛡️ Prevention

### Primary Defenses

- Normalize all token amounts to a consistent internal precision (e.g., 18 decimals).
- Use IERC20Metadata(token).decimals() during setup or token registration.
- Never assume a token uses 18 decimals without explicit validation.

### Additional Safeguards

- Include tests with common tokens like USDC (6 decimals), DAI (18), and wBTC (8).
- Add assertions or guards to prevent overflow during normalization.
- Validate external protocol integrations for expected precision.

### Detection Methods

- Slither: decimal-assumption, precision-inconsistency, unsafe-token-math detectors.
- Manual audit of all logic involving amount * 1e18 or division by 1e18.
- Integration testing with tokens using varied decimal schemes.

## 🕰️ Historical Incidents

- **Name:** Curve Metapool Decimal Mismatch 
- **Date:** 2021 
- **Impact:** Unexpected pricing behavior due to USDC 6-decimal conversion 
- **Post-mortem:** [Link](https://curve.fi) 


## 📚 Further Reading

- [SWC-136: Unexpected Behavior from Environmental Assumptions](https://swcregistry.io/docs/SWC-136) 
- [Solidity Docs – Fixed Point Math Considerations](https://docs.soliditylang.org/en/latest/) 
- [OpenZeppelin IERC20Metadata](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Metadata) 


---

## ✅ Vulnerability Report 

```YAML
id: TBA
title: Inconsistent Decimal Precision 
severity: M
score:
impact: 4         
exploitability: 2 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.4

```

---

## 📄 Justifications & Analysis

- **Impact**: Can corrupt accounting and economic logic across protocols.
- **Exploitability**: Not exploitable by attacker alone but can amplify other issues.
- **Reachability**: Common when using USDC, USDT, wBTC with 6–8 decimals in 18-decimal logic.
- **Complexity**: Fixable with clear normalization functions.
- **Detectability**: High — test and audit patterns reveal these errors easily.