# Dangerous Unary Expressions

```YAML
id: TBA
title: Dangerous Unary Expressions Cause Incorrect State Mutations and Logic Errors
severity: M
category: logic
language: solidity
blockchain: [ethereum]
impact: Wrong calculations, state overwrite, or logic bypass
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-682
swc: SWC-135
```

## 📝 Description

- In Solidity, unary expressions like ++, --, -, and ! can behave unexpectedly when:
- Used within inline statements
- Misordered relative to assignments
- Applied to uint types, causing silent underflow in older versions (<0.8.0)
- In particular, confusing use of ++var vs var++ and mixing assignment and side effects in one line can lead to incorrect state changes. If these expressions are used without clarity or boundaries, they may:
- Cause off-by-one errors
- Increment or decrement the wrong value
- Overwrite important mappings or state variables incorrectly

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Counter {
    mapping(address => uint256) public counts;

    function logAndIncrement() external {
        counts[msg.sender] = counts[msg.sender]++;
        // ❌ Value is not incremented as expected
    }
}
```

## 🧪 Exploit Scenario

1. A staking contract logs participation by incrementing a user count.
2. It uses storage[addr] = storage[addr]++ as shorthand.
3. The ++ is applied but its value is lost due to reassignment.
4. User staking count remains stuck at 0.
5. Rewards, permissions, or unlocks based on stake count never trigger, locking users out or enabling repeat rewards due to failure to advance state.

**Assumptions:**

- The logic uses ++ or -- in expressions with assignment or mapping access.
- There is no validation or test to detect silent failures.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeCounter {
    mapping(address => uint256) public counts;

    function logAndIncrement() external {
        counts[msg.sender] += 1; // ✅ Clear and correct
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid combining ++ or -- with assignment (x = x++).
- Use += 1 or -= 1 to make intent explicit.

### Additional Safeguards

- Add test cases that assert variable changes across all mutations.
- Avoid in-line increment/decrement inside expressions that also mutate state.

### Detection Methods

- Search for lines containing both = and ++ or --.
- Tools: Slither (dangerous-unary), Solhint (no-unchecked-unary-expressions), manual review

## 🕰️ Historical Exploits

- **Name:** Solidity Unary Plus Operator Misuse 
- **Date:** 2017 
- **Loss:** Potential for unintended behavior due to misuse of unary plus operator 
- **Post-mortem:** [Link to post-mortem](https://github.com/ethereum/solidity/issues/1760)
 
## 📚 Further Reading

- [SWC-135: Incorrect Logic](https://swcregistry.io/docs/SWC-135/) 
- [Solidity Docs – Operators](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#operators)
- [Slither – Expression Order Pitfalls](https://github.com/crytic/slither/wiki/Detector-Documentation#post-increment-in-assignments) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Dangerous Unary Expressions Cause Incorrect State Mutations and Logic Errors
severity: M
score:
impact: 3        
exploitability: 2 
reachability: 4   
complexity: 1    
detectability: 5 
finalScore: 2.9
```

---

## 📄 Justifications & Analysis

- **Impact**: State may be mutated incorrectly or not at all, blocking flows or enabling unintended behavior.
- **Exploitabilit**y: Primarily internal misuse; users can't exploit it directly, but may benefit indirectly (e.g., repeat access).
- **Reachability**: Occurs in many contract types—lotteries, staking, counters.
- **Complexity**: A very simple syntax error with high consequence.
- **Detectability**: Readily caught by Slither or manual logic testing.