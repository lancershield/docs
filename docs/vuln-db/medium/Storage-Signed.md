# Storage Signed Integer 

```YAML
id: TBA
title: Storage Signed Integer 
baseSeverity: M
category: storage
language: solidity
blockchain: [ethereum]
impact: Incorrect accounting, overflow/underflow, or logic corruption
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-190
swc: SWC-101
```

## 📝 Description

- When using signed integer arrays (e.g., int256[]) in storage, developers often assume they behave the same as uint[] in terms of safety and overflow handling. 
- However, if unguarded arithmetic is performed—especially with older Solidity versions that lack SafeMath for int—this can lead to:
- Silent overflows and underflows.
- Negative values being used where only positive ones are expected (e.g., balances, indexes).
- Misaligned logic in deposit/withdrawal systems, voting, or reward tracking.
- Incorrect assumptions about sign, bounds, or unchecked math can destabilize contract behavior.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract SignedArray {
    int256[] public balances;

    function deposit(int256 amount) external {
        require(amount > 0, "must deposit positive");
        balances.push(amount);
    }

    function withdraw(uint index, int256 amount) external {
        balances[index] -= amount; // ❌ Underflow risk if amount > balances[index]
    }
}
```

## 🧪 Exploit Scenario

1. A user deposits 10 units → balances[0] = 10.
2. They later call withdraw(0, 20).
3. The contract subtracts 20 from 10, resulting in -10.
4. If this value is later used in logic expecting positive balances (e.g., rewards, governance weight), it breaks invariants or causes logic failure.

**Assumptions:**

- The contract uses int256[] instead of uint[].
- No explicit bounds or sign checks are in place before arithmetic operations.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeSignedArray {
    int256[] public balances;

    function deposit(int256 amount) external {
        require(amount > 0, "must deposit positive");
        balances.push(amount);
    }

    function withdraw(uint index, int256 amount) external {
        require(amount > 0, "invalid withdraw amount");
        require(balances[index] >= amount, "insufficient balance");

        balances[index] -= amount; // ✅ Underflow prevented
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Signed storage misuse leads to logic bugs and underflows."
- context: "DeFi protocol managing collateral and debt"
  severity: H
  reasoning: "Debt-based systems relying on signed math may grant unearned access or rewards."
- context: "Token contract with restricted mint/burn rules"
  severity: L
  reasoning: "Risk mitigated due to fewer signed variables and tighter controls."
```

## 🛡️ Prevention

### Primary Defenses

- Avoid using int256[] unless negative values are essential.
- Validate all arithmetic conditions to avoid underflows and logic errors.

### Additional Safeguards

- Use custom SafeMath-style libraries for signed integers if pre-0.8.0.
- Document semantics clearly when using signed arrays for edge cases.
- Test negative paths and edge cases explicitly in unit tests.

### Detection Methods

- Scan for int[] or int256[] used in storage and arithmetic operations.
- Look for unchecked subtraction or multiplication logic.
- Tools: Slither (unchecked-arithmetic, signed-arithmetic), MythX, manual audit

## 🕰️ Historical Exploits

- **Name:** Solidity Storage Array Bug 
- **Date:** 2019 
- **Loss:** Potential data corruption in contracts using signed integer arrays 
- **Post-mortem:** [Link to post-mortem](https://soliditylang.org/blog/2019/06/25/solidity-storage-array-bugs/) 

## 📚 Further Reading

- [SWC-101: Integer Overflow and Underflow](https://swcregistry.io/docs/SWC-101/) 
- [Solidity Docs – Arithmetic and Overflow](https://docs.soliditylang.org/en/latest/control-structures.html#checked-or-unchecked-arithmetic) 
- [OpenZeppelin SafeMath for Signed Ints](https://docs.openzeppelin.com/contracts/2.x/api/math#SignedSafeMath) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Storage Signed Integer 
severity: M
score:
impact: 3         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 3.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Signed underflow may result in negative balances or broken logic.
- **Exploitability**: Attacker can manipulate inputs to create invalid state.
- **Reachability**: Common when developers use int256[] for tracking balances or deltas.
- **Complexity**: Low mistake, but requires understanding of signed math edge cases.
- **Detectability**: Missed when relying on compiler overflow protections alone.


