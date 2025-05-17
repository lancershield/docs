# Deprecated Solidity Functions 


```YAML
id: TBA
title: Deprecated Solidity Functions Leading to Undefined Behavior or Upgrade Failures
severity: M
category: language-usage
language: solidity
blockchain: [ethereum]
impact: Execution inconsistencies, forward-compatibility issues, or security flaws
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-676
swc: SWC-130
```

## 📝 Description

- Deprecated Solidity functions refer to outdated or removed language constructs and standard functions that are no longer recommended for use due to behavioral inconsistencies, security vulnerabilities, or compatibility problems. These include:
- `sha3()` (alias for `keccak256()`)
- `throw` (replaced by `revert`)
- `suicide` (replaced by `selfdestruct`)
- `constant` (replaced by `view` or `pure`)
- `now` (replaced by `block.timestamp`)
- `var` (replaced by explicit type declaration)
- Using these deprecated features can:
- Cause unexpected behavior on newer compilers,
- Break upgrade paths, or
- Trigger audit tool warnings and forward compatibility issues.

## 🚨 Vulnerable Code

```solidity
contract LegacyContract {
    function hashData(bytes memory data) public pure returns (bytes32) {
        return sha3(data); // ❌ Deprecated; use keccak256 instead
    }

    function kill() public {
        if (msg.sender != owner) throw; // ❌ Deprecated; use require/revert
        suicide(owner); // ❌ Deprecated; use selfdestruct
    }
}
```

## 🧪 Exploit Scenario

1. While not always directly exploitable, the use of deprecated functions may:
2. Break compatibility with modern tools, auditors, or compiler targets.
3. Introduce subtle logic bugs due to syntax ambiguity.
4. Be mistakenly used in contexts that behave differently today (e.g., now → block.timestamp returns unexpected values in future forks).

**Assumptions:**

- Developers have not refactored legacy constructs.
- Contracts are compiled or upgraded without proper testing.

## ✅ Fixed Code

```solidity

contract ModernContract {
    function hashData(bytes memory data) public pure returns (bytes32) {
        return keccak256(data); // ✅ Modern hashing function
    }

    function kill() public {
        require(msg.sender == owner, "Unauthorized");
        selfdestruct(payable(owner));
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always use the latest Solidity version (or one with critical updates like 0.8.x+).
- Replace deprecated terms with their secure alternatives (keccak256, revert, selfdestruct, etc.).
- Avoid var, throw, sha3, suicide, and now in all new code.

### Additional Safeguards

- Use solhint, solc warnings, and Slither to detect deprecated syntax.
- Run unit tests after compiler upgrades to catch legacy logic changes.
- Apply explicit visibility, return types, and safe fallback mechanisms.

### Detection Methods

- Slither: deprecated-constructs, legacy-function-use, sha3-detected, throw-detected rules.
- solc compiler warnings (enable --allow-paths and --via-ir for better detection).
- Static code linters like solhint, Solium, or hardhat plugins.

## 🕰️ Historical Exploits

 - **Name:** Proof of Weak Hands Coin (PoWHCoin) Exploit 
 - **Date:** 2018-03-09 
 - **Loss:** Approximately 866 ETH 
 - **Post-mortem:** [Link to post-mortem](https://101blockchains.com/integer-overflow-attacks/) 
  

## 📚 Further Reading

- [SWC-130: Deprecated/Unused Code](https://swcregistry.io/docs/SWC-130) 
- [Solidity Docs – Breaking Changes](https://docs.soliditylang.org/en/latest/080-breaking-changes.html) 
- [Slither Deprecated Constructs Detector](https://github.com/crytic/slither) - [Solhint Rule Set](https://protofire.github.io/solhint/) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Deprecated Solidity Functions Leading to Undefined Behavior 
severity: M
score:
impact: 3         
exploitability: 2 
reachability: 5   
complexity: 1     
detectability: 5  
finalScore: 3.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Causes long-term security and compatibility issues, especially during upgrades or forks.
- **Exploitability**: Unlikely in isolation but can contribute to larger flaws.
- **Reachability**: Common in older codebases, often reused.
- **Complexity**: Simple mistakes or legacy patterns reintroduced.
- **Detectability**: High — modern tools highlight deprecated usage clearly.