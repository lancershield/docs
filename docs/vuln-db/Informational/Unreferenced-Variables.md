# Unreferenced Variables

```YAML
id: LS05I
title:
Unreferenced Variables
baseSeverity: I
category: code-quality
language: solidity
blockchain: [ethereum]
impact: Unused storage or memory increases bytecode size and audit complexity
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.24"]
cwe: CWE-563
swc: SWC-131
```

## 📝 Description

- Unreferenced or unused variables are declared but never read or written after their initial declaration. 
- While they don’t pose direct security threats, they clutter the codebase, increase contract bytecode size, and potentially mislead auditors or developers during review. 
- When declared in storage, these variables also consume additional deployment gas and persistent space on-chain.
- This issue can lead to:
- Miscommunication about contract logic intent
- Increased audit surface

## 🚨 Vulnerable Code

```solidity

contract UnusedStorage {
    uint256 public unusedStorage; // never referenced
    function setSomething() external {
        uint256 unusedLocal = 42; // declared but never used
        // no logic using unusedLocal
    }
}
```

## 🧪 Exploit Scenario

Step-by-step impact sequence:

1. Developer declares a storage variable (unusedStorage) that is never read or updated.
2. Users interact with the contract unaware of the variable's redundancy.
3. Contract consumes additional gas during deployment due to unused storage slots.
4. Auditor misinterprets unused variables as integral to contract logic, possibly missing real bugs.

**Assumptions:**

- The variable is declared in storage or memory.
- There are no code optimizations to remove it during compilation.
- Developers and auditors might assume functionality that isn’t actually implemented.

## ✅ Fixed Code

```solidity

contract CleanedStorage {
    // unusedStorage removed

    function setSomething() external {
        // unusedLocal removed
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "No functional impact, but bloats contract and increases audit burden."
- context: "Gas-sensitive microcontracts (e.g., minimal vaults)"
  severity: M
  reasoning: "Unnecessary gas usage and size bloating undermines optimization."
- context: "Verified production contracts on-chain"
  severity: I
  reasoning: "No runtime impact, just minor cosmetic and storage issues."
```

## 🛡️ Prevention

### Primary Defenses

- Use static analyzers like Slither to catch unreferenced variables before deployment.
- Adopt strict linting rules during CI (e.g., solhint no-unused-vars).

### Additional Safeguards

- Perform code review with emphasis on logic-to-variable relevance.
- Document variable purpose explicitly using NatSpec or inline comments.

### Detection Methods

- Slither Flags unused-state, unused-local.
- MythX Includes unused variable detection in standard security profiles.
- Manual Review Look for variables not present in any execution path.

## 🕰️ Historical Exploits

- **Name:** YAM Finance Rebase Bug
- **Date:** August 2020
- **Loss:** ~$750,000
- **Post-mortem:** [Link to post-mortem](https://medium.com/certik/2020-08-13-yam-finance-smart-contract-bug-analysis-future-prevention-b4220976ebea)

## 📚 Further Reading

- [SWC-131: Unused State Variables – SWC Registry](https://swcregistry.io/docs/SWC-131/)
- [Slither: unused-state and unused-local detectors](https://github.com/crytic/slither)
- [CWE-563: Unused Variable – MITRE](https://cwe.mitre.org/data/definitions/563.html) 

---

## ✅ Vulnerability Report

```markdown
id: LS05I
title: Unreferenced Variables
severity: I
score:
impact: 1     
exploitability: 0 
reachability: 3  
complexity: 1     
detectability: 5  
finalScore: 1.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Does not change or endanger contract logic or funds.
- **Exploitability**: No exploitable vector; purely quality-related.
- **Reachability**: Common in large codebases or during rapid development.
- **Complexity**: Trivial to understand and eliminate.
- **Detectability**: Easily caught by all major static analysis tools.

