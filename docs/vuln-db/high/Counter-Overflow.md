# Counter Overflow 

```YAML
id: LS14H
title: Counter Overflow 
baseSeverity: H
category: arithmetic
language: solidity
blockchain: [ethereum]
impact: Loop execution, minting, or identifier logic may overflow and break constraints
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.0"]
cwe: CWE-190
swc: SWC-101
```

## 📝 Description

- Counter overflow occurs when a counter variable (typically `uint256` or `uint8`) increments past its maximum value and wraps around to 0. Prior to Solidity 0.8.0, arithmetic operations do not revert on overflow, making it possible for:
- Loop bounds to break,
- Counters to reset,
- ID values to duplicate or become unpredictable.
- This can lead to minting duplicate tokens, infinite loops, or violations of uniqueness guarantees, especially in NFT, vault, and staking logic.

## 🚨 Vulnerable Code

```solidity
contract VulnerableCounter {
    uint8 public counter;

    function increment() public {
        counter += 1; // ❌ Overflow not checked in Solidity <0.8.0
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Contract uses a uint8 counter to assign token IDs or loop iteration.
2. Attacker increments the counter 256 times.
3. Counter wraps back to 0 (255 + 1 == 0) due to overflow.
4. Token ID 0 or index 0 is reused or minted again, breaking uniqueness logic.

**Assumptions:**

- No overflow checks (i.e., Solidity <0.8.0).
- Counter is small (uint8, uint16, etc.).
- Used in critical indexing or iteration logic.

## ✅ Fixed Code

```solidity
pragma solidity ^0.8.0;

contract SafeCounter {
    uint256 public counter;

    function increment() public {
        require(counter < type(uint256).max, "Overflow");
        counter += 1;
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Unique ID generation, token minting, or voting"
  severity: H
  reasoning: "Can cause duplicated identities, logic corruption, or stolen assets"
- context: "Gas metering or unused counter with no impact on access or storage"
  severity: L
  reasoning: "Wraparound has no critical effect on protocol logic"
- context: "Contract uses Solidity ≥0.8.0 with overflow protection"
  severity: I
  reasoning: "No issue present due to native safety checks"
```

## 🛡️ Prevention

### Primary Defenses

- Use Solidity >=0.8.0, which automatically reverts on overflow.
- Use OpenZeppelin Counters library for ID tracking and safe iteration.
- Avoid small-width types (uint8, uint16) unless justified for optimization.

### Additional Safeguards

- Enforce explicit upper bounds with require(counter < maxLimit).
- Write fuzz tests for counter rollovers.
- Monitor contract state to detect wrap-around conditions.

### Detection Methods

- Slither: unchecked-arithmetic, counter-overflow, increment-overflow detectors.
- Manual audit of all loop counters, token ID generators, and accumulators.
- Hardhat + Foundry fuzz tests for maximum-bound edge cases.

## 🕰️ Historical Exploits

- **Name:** BeautyChain (BEC) Token Overflow Exploit 
- **Date:** 2018 
- **Loss:** Massive token inflation leading to value collapse 
- **Post-mortem:** [Link to post-mortem](https://smartstate.tech/blog/understanding-overflow-and-underflow-vulnerabilities-in-smart-contracts.html)


## 📚 Further Reading

- [SWC-101: Integer Overflow and Underflow](https://swcregistry.io/docs/SWC-101) 
- [OpenZeppelin Counters Library](https://docs.openzeppelin.com/contracts/4.x/api/utils#Counters) 
- [Solidity Docs – Arithmetic Overflow](https://docs.soliditylang.org/en/v0.8.0/080-breaking-changes.html) 

---

## ✅ Vulnerability Report

```markdown
id: LS14H
title: Counter Overflow 
severity: H
score:
impact: 5         
exploitability: 3
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Serious issues like ID duplication or protocol logic failure.
- **Exploitability**: May require repeated calls or long execution.
- **Reachability**: Found anywhere counters are used without upper bounds.
- **Complexity**: Simple mistake — especially dangerous with small uint sizes.
- **Detectability**: Readily detectable with modern tools or language upgrades.