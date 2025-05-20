# Incorrect Exponentiation Causes Logic Errors and Fund Miscalculation

```YAML
id: TBA
title: Incorrect Exponentiation Causes Logic Errors and Fund Miscalculation
severity: H
category: arithmetic
language: solidity
blockchain: [ethereum]
impact: Financial miscalculation, rewards distortion, or balance drift
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-682
swc: SWC-128
```

## 📝 Description

- In Solidity, exponentiation (**) operates only on integer types and follows truncation rules—meaning floating-point behavior is not supported. 
- When exponentiation is used incorrectly, particularly in percentage calculations, APR/APY logic, or token reward formulas, it can produce inaccurate results due to:
- Integer rounding (e.g., 1.1 truncated to 1)
- Misuse of base/exponent scale (e.g., using ^ instead of scaling with wad/ray)
- Incorrect reward growth logic (e.g., base ** time instead of using compounding formula)
- These mistakes often go unnoticed until reward pools accumulate losses or user balances drift over time.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract IncorrectPower {
    function computeReward(uint256 base, uint256 years) public pure returns (uint256) {
        return base ** years; // ❌ Incorrect if base is < 1 or not scaled properly
    }
}
```

## 🧪 Exploit Scenario

1. A staking contract tries to reward users with compound interest using base ** time.
2. The base is derived from a scaled decimal like 1.05 * 1e18.
3. Due to integer-only behavior, base ** time = 1 ** time = 1.
4. Users receive no rewards, or worse, receive flat or incorrect payouts.
5. Attackers may deposit large sums to exploit a broken reward curve and extract excessive or zeroed rewards.

**Assumptions:**

- Developer assumes exponentiation behaves like floating-point math.
- Reward calculations, growth curves, or APY logic depend on this behavior.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

library Math {
    function pow(uint256 base, uint256 exp, uint256 scale) internal pure returns (uint256) {
        uint256 result = scale;
        for (uint256 i = 0; i < exp; i++) {
            result = result * base / scale;
        }
        return result;
    }
}

contract CorrectPower {
    uint256 constant SCALE = 1e18;

    function computeReward(uint256 base, uint256 years) public pure returns (uint256) {
        return Math.pow(base, years, SCALE);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Never use ** with decimal values or scaled integers directly.
- Use fixed-point exponentiation functions from vetted math libraries.

### Additional Safeguards

- Apply test cases with known exponent results to verify output.
- Avoid implementing exponential growth in Solidity unless necessary.
- Use off-chain computation for complex financial formulas.

### Detection Methods

- Look for use of ** in any economic logic.
- Review staking, bonding, or rewards logic for reliance on **.
- Tools: Slither (incorrect-exponentiation custom rule), manual audit

## 🕰️ Historical Exploits

- **Name:** Olympus DAO Early Bonding Curve Bugs 
- **Date:** 2021 
- **Loss:** Price drift, fixed in bonding math 
- **Post-mortem:** [Link to post-mortem](https://github.com/OlympusDAO/olympus-contracts/issues) 
  

## 📚 Further Reading

- [SWC-128: DoS with Block Gas Limit](https://swcregistry.io/docs/SWC-128/) 
- [Solidity Docs – Exponentiation Operator](https://docs.soliditylang.org/en/latest/types.html#exponentiation) 
- [PRBMath – High-Precision Math Library](https://github.com/PaulRBerg/prb-math) 
- [ABDKMath64x64 – Fixed Point Math](https://github.com/abdk-consulting/abdk-libraries-solidity) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Incorrect Exponentiation Causes Logic Errors and Fund Miscalculation
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 3   
complexity: 3     
detectability: 3  
finalScore: 3.4
```

## 📄 Justifications & Analysis

- **Impact**: Incorrect rewards may cause over- or under-distribution of assets.
- **Exploitability**: Attackers can abuse non-linear behavior to game returns.
- **Reachability**: Seen in any project using APR/APY with staking or time-based rewards.
- **Complexity**: Misunderstood behavior of ** with decimals or scaled values.
- **Detectability**: Missed without unit tests or financial accuracy checks.
