# Builtin Symbol Shadowing

```YAML
id: TBA
title: Builtin Symbol Shadowing Introduces Hidden Bugs and Misleading Logic
severity: L
category: naming
language: solidity
blockchain: [ethereum]
impact: Confusing behavior, incorrect assumptions, or broken function calls
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-563
swc: SWC-135
```

## 📝 Description

- Solidity has several built-in global symbols such as block, msg, tx, now (deprecated), super, and this.
- Accidentally shadowing these symbols by declaring variables or functions with the same name can result in:
- Unexpected behavior or execution failures
- Confusing or misleading logic
- Hidden bugs in require(), revert(), or external calls
- Because built-ins are part of the global scope, shadowing them hides the original meaning in the current scope. Static analyzers may not always flag this issue unless explicitly configured to detect name collisions.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract SymbolShadowing {
    function testBlock(uint256 block) public pure returns (uint256) {
        return block.timestamp;  // ❌ Error: `block` no longer refers to the global `block`
    }
}
```

## 🧪 Exploit Scenario

1. A developer writes a utility function function getTx(), unintentionally shadowing tx.
2. Elsewhere in the contract, tx.origin is used, but due to a naming collision with getTx, a different value is returned or the compiler emits a misleading error.
3. The deployed logic behaves differently than intended, potentially skipping validations or creating incorrect conditions for authorization.

**Assumptions:**

- Symbol shadowing occurs in a function parameter, local variable, or contract-level declaration.
- Developers reference the shadowed symbol elsewhere, assuming global behavior.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeNaming {
    function testBlock(uint256 blockNumber) public view returns (uint256) {
        return block.timestamp;  // ✅ Safe: no shadowing
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid using names that match Solidity built-ins or global objects.
- Use linting tools with shadowing detection enabled (e.g., solhint, solc --ast).

### Additional Safeguards

- Apply naming conventions such as prefixing parameters (_param) to reduce collision risk.
- Peer review and audit functions that reference msg, block, tx, etc.

### Detection Methods

- Look for variables or parameters named block, msg, tx, this, now, or super.
- Tools: Slither (shadowing), Solhint (no-shadow-restricted-names), manual review

## 🕰️ Historical Exploits

- **Name:** Modifier Shadowing Built-in `require` 
- **Date:** 2019 
- **Loss:** Potential misbehavior due to overriding the built-in `require` function 
- **Post-mortem:** [Link to post-mortem](https://ethereum.stackexchange.com/questions/77193/warning-this-declaration-shadows-a-builtin-symbol) 
- **Name:** State Variable Shadowing in Inherited Contracts 
- **Date:** 2022 
- **Loss:** Unexpected behavior due to shadowed state variables in inheritance 
- **Post-mortem:** [Link to post-mortem](https://medium.com/@joichiro.sai/solidity-attack-vector-25-shadowed-state-variables-c5ccc2944c16) 
  

📚 Further Reading

- [SWC-135: Incorrect Logic](https://swcregistry.io/docs/SWC-135/) 
- [Slither – Shadowing Detection](https://github.com/crytic/slither/wiki/Detector-Documentation#variable-shadowing)
- [Solidity Naming Conventions](https://docs.soliditylang.org/en/latest/style-guide.html) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Builtin Symbol Shadowing Introduces Hidden Bugs and Misleading Logic
severity: L
score:
impact: 2  
exploitability: 1 
reachability: 3   
complexity: 1     
detectability: 5  
finalScore: 2.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Causes misbehavior or runtime bugs by overriding Solidity’s global variables.
- **Exploitability**: Not externally triggered; results from developer confusion.
- **Reachability**: Common in utility functions, especially those written quickly or ported.
- **Complexity**: Trivial bug; easy to introduce and resolve.
- **Detectability**: Highly detectable via linting and naming rules.