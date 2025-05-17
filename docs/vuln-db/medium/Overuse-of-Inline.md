# Overuse of Inline Assembly

```YAML
id: TBA
title: Overuse of Inline Assembly 
severity: M
category: code-quality
language: solidity
blockchain: [ethereum]
impact: Increased risk of undetected bugs, misbehavior, or low-level exploits
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<latest"]
cwe: CWE-489
swc: SWC-134
```

## 📝 Description

- Overuse of inline assembly (`assembly { ... }`) in Solidity contracts introduces low-level EVM logic that bypasses type safety, overflow checks, and compiler optimizations. 
- While sometimes justified for gas optimization or raw memory access, excessive or unjustified assembly usage can:
- Obscure business logic and reduce auditability
- Introduce undefined or unsafe behavior (e.g., memory corruption, storage collision)
- Break forward compatibility with newer Solidity versions,
- Allow security vulnerabilities to go unnoticed by static analysis tools.

## 🚨 Vulnerable Code

```solidity
contract UnsafeAssembly {
    function unsafeAdd(uint256 a, uint256 b) public pure returns (uint256 result) {
        assembly {
            result := add(a, b) // ❌ No overflow check, no input validation
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step impact:

1. Developer uses assembly to save gas on basic arithmetic or logic.
2. Solidity version is <0.8.0, meaning no built-in overflow checks.
3. add(a, b) overflows silently (e.g., 2**256 - 1 + 1 = 0).
4. User receives incorrect results or protocol logic behaves unpredictably.

**Assumptions:**

- Inline assembly is used where standard Solidity would suffice.
- No validation or fallback mechanisms are in place.

## ✅ Fixed Code

```solidity

contract SafeMath {
    function safeAdd(uint256 a, uint256 b) public pure returns (uint256) {
        return a + b; // ✅ Solidity handles overflow in 0.8.0+
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid inline assembly unless strictly necessary (e.g., hashing, gas optimizations).
- Use Solidity’s built-in arithmetic, memory, and storage operations when available.
- If assembly is required:
- Isolate it in well-documented internal functions.
- Validate all inputs and outputs.
- Use comments to explain why and what the assembly block does.

### Additional Safeguards

- Limit assembly to audited libraries (e.g., OpenZeppelin’s low-level utilities).
- Annotate assembly blocks with compiler assumptions (solc version, memory layout).

- inline assembly paths with unit tests and fuzzers.

### Detection Methods

- Slither: assembly-usage, dangerous-inline-assembly, low-level-memory-access detectors.
- Manual audit of all assembly {} blocks for bounds checks and memory alignment.
- Coverage tools to ensure assembly paths are tested.

## 🕰️ Historical Exploits

- **Name:** Parity Wallet Selfdestruct Bug 
- **Date:** 2017 
- **Loss:** ~$150M frozen 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert-2/) 


## 📚 Further Reading

- [SWC-134: Message call with hardcoded gas amount](https://swcregistry.io/docs/SWC-134) 
- [Solidity Docs – Inline Assembly](https://docs.soliditylang.org/en/latest/assembly.html) 
- [Slither – Assembly Usage Detector](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Overuse of Inline Assembly 
severity: M
score:
impact: 3         
exploitability: 2 
reachability: 4   
complexity: 4     
detectability: 4  
finalScore: 3.3
```

---

## 📄 Justifications & Analysis

- **Impact**: May lead to critical misbehavior, especially in math, memory, or proxy contexts.
- **Exploitability**: Requires the presence of unsafe assembly logic that can be triggered or bypassed.
- **Reachability**: Common in custom low-level libraries or gas-optimized functions.
- **Complexity**: High — often written without documentation and bypasses Solidity's safety net.
- **Detectability**: High — flagged by tools like Slither and easily reviewed when isolated.
