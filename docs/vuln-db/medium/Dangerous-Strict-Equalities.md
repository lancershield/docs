# Dangerous Strict Equalities 

```YAML
id: LS14M
title: Dangerous Strict Equalities 
baseSeverity: M
category: logic
language: solidity
blockchain: [ethereum]
impact: Logic bypass, unintended execution paths, or bricked flows
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-697
swc: SWC-135
```

## 📝 Description

- Using strict equality comparisons (==) on dynamic or user-controlled values—such as token balances, timestamps, or return values—can lead to brittle and incorrect logic, especially in the presence of rounding errors, asynchronous state changes, or multi-actor conditions.
- This becomes particularly dangerous when:
- Comparing token.balanceOf(...) == expectedAmount
- Checking block.timestamp == expiryTime
- Verifying precise collateral ratios, votes, or slot positions
- Small deviations can cause the contract to enter unintended branches or permanently lock logic (e.g., funds stuck due to missing an exact condition).

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract FragileVault {
    mapping(address => uint256) public deposits;

    function withdraw(uint256 amount) external {
        require(deposits[msg.sender] == amount, "Amount mismatch"); // ❌ Strict equality
        deposits[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

## 🧪 Exploit Scenario

1. A user deposits 100.000000000000000001 ETH.
2. UI rounds display to 100 ETH, and user tries to withdraw 100.
3. require(deposits[msg.sender] == 100 ether) fails due to strict equality.
4. User cannot recover funds unless they perfectly match the internal storage.

**Assumptions:**

- Logic relies on exact matches for values influenced by external systems (UI, rounding, time).
- No tolerance or fallback logic is included.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeVault {
    mapping(address => uint256) public deposits;

    function withdraw(uint256 amount) external {
        require(deposits[msg.sender] >= amount, "Insufficient balance"); // ✅ Safer comparison
        deposits[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: M
  reasoning: "Logic failure without financial theft, but may block contract use."
- context: "DeFi claim logic or withdrawal unlock conditions"
  severity: H
  reasoning: "Funds may be permanently locked or inaccessible."
- context: "Internal DAO setting or testnet use"
  severity: L
  reasoning: "Impact limited due to access controls or testing scope."
```

## 🛡️ Prevention

### Primary Defenses

- Avoid == for comparisons involving balances, timestamps, percentages, or calculations.
- Use safe inequalities or bounded ranges (x >= y, abs(x - y) <= epsilon).

### Additional Safeguards

- Normalize or floor values before comparing (e.g., fixed-point rounding).
- Add slippage/tolerance parameters for user-supplied values.

### Detection Methods

- Scan for == operators used on balanceOf, timestamp, or computed values.
- Flag equality checks that don't account for variance or rounding.
- Tools: Slither (dangerous-comparison), manual review of conditionals

## 🕰️ Historical Exploits

- **Name:** MakerDAO Oracle Discrepancy 
- **Date:** 2020 
- **Loss:** None (issue detected in testnet) 
- **Post-mortem:** [Link to post-mortem](https://forum.makerdao.com/) 

## 📚 Further Reading

- [SWC-135: Incorrect Authorization (includes logic flaws)](https://swcregistry.io/docs/SWC-135/) 
- [Solidity Docs – Comparison Operators](https://docs.soliditylang.org/en/latest/control-structures.html#comparison-operators)
- [ChainSecurity Audit Notes on Strict Equalities](https://chainsecurity.com/) 

---

## ✅ Vulnerability Report

```markdown
id: LS14M
title: Dangerous Strict Equalities 
severity: M
score:
impact: 3         
exploitability: 3 
reachability: 4   
complexity: 2    
detectability: 4  
finalScore: 3.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Misplaced strict equality blocks user action or misroutes state transitions.
- **Exploitability**: Trivially reproducible through UI rounding or crafted calldata.
- **Reachability**: Found in common flows like withdraw, claim, redeem, update.
- **Complexity**: Low sophistication mistake—easy to introduce.
- **Detectability**: Often missed unless comparison expressions are manually reviewed.
