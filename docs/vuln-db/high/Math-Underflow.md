# Math Underflow in Loops

```YAML
id: TBA
title: Math Underflow in Loops Leading to Infinite Iteration or Logic Corruption
severity: H
category: arithmetic
language: solidity
blockchain: [ethereum]
impact: Infinite loop, denial of service, or skipped logic execution
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.0"]  # Note: Solidity 0.8.0+ has built-in overflow checks
cwe: CWE-682
swc: SWC-101
```

## 📝 Description

- Math underflow in loops occurs when a loop counter or index variable decrements below zero due to incorrect conditions or arithmetic. 
- In Solidity versions prior to 0.8.0, underflows do not revert by default, and `uint` types wrap around (i.e., `0 - 1 == 2^256 - 1`). This can result in:
- Infinite loops consuming all gas,
- Skipped logic execution due to improper indexing,
- DoS vectors on shared state due to reentrancy or miscounts.

## 🚨 Vulnerable Code

```solidity
contract UnderflowLoop {
    uint256[] public data;

    function clear() public {
        // ❌ i underflows from 0 to 2^256 - 1, causing infinite loop
        for (uint256 i = data.length - 1; i >= 0; i--) {
            data.pop();
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. data.length == 0, and user calls clear().
2. i = 0 - 1 wraps to 2^256 - 1 (uint underflow).
3. Loop never terminates, consuming all gas.
4. On shared contracts, this can block governance actions, staking withdrawals, or refund claims.

**Assumptions:**

- Loop uses unchecked decrement (i--) with uint type.
- Solidity version <0.8.0 (no built-in SafeMath).
- No bounds check or loop condition sanitization.

## ✅ Fixed Code

```solidity
contract SafeLoop {
    uint256[] public data;

    function clear() public {
        uint256 len = data.length;
        if (len == 0) return;

        for (uint256 i = len; i > 0; i--) {
            data.pop(); // ✅ Safe: i goes from len to 1
        }
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Upgrade to Solidity >=0.8.0 where overflows/underflows are checked by default.
- Use i > 0; i-- loop bounds instead of i >= 0, and access data[i - 1].
- Use unchecked { i-- } only when safe and bounds are manually verified.

### Additional Safeguards

- Consider using while loops for better clarity in reverse iteration.
- Implement invariant checks using assertions (e.g., assert(i < max)).
- Write fuzz tests that simulate boundary conditions (empty array, single entry, max size).

### Detection Methods

- Slither: loop-counter-underflow, unchecked-arithmetic, dangerous-loop detectors.
- Manual review of all for (...) loops with decrementing counters.
- Unit tests with empty input conditions and low loop ranges.

## 🕰️ Historical Exploits

- **Name:** DecentralizedBank Underflow Exploit 
- **Date:** 2024-01 
- **Loss:** 5 ETH 
- **Post-mortem:** [Link to post-mortem](https://www.kayssel.com/post/web3-8/)

## 📚 Further Reading

- [SWC-101: Integer Overflow and Underflow](https://swcregistry.io/docs/SWC-101)
- [Solidity Docs – Looping Constructs](https://docs.soliditylang.org/en/latest/control-structures.html#loops) 
- [Trail of Bits – Solidity Gotchas](https://github.com/crytic/awesome-ethereum-security) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Math Underflow in Loops Leading to Infinite Iteration or Logic Corruption
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Infinite loops or skipped logic lead to denial of service or protocol lockups.
- **Exploitability**: Triggered by edge-case inputs (e.g., empty arrays).
- **Reachability**: Found in many DAO, airdrop, or batch processing patterns.
- **Complexity**: Low — just requires triggering a bad loop state.
- **Detectability**: Easily flagged by tools or test coverage with boundary values.