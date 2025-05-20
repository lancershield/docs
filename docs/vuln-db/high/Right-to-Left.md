# Right-to-Left Override (RTLO) Character Obfuscation Enables Malicious File or Variable Masking

```YAML
id: TBA
title: Right-to-Left Override (RTLO) Character Obfuscation Enables Malicious File or Variable Masking
severity: H
category: obfuscation
language: solidity
blockchain: [ethereum]
impact: Audit evasion, variable misdirection, or malicious behavior obfuscation
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-779
swc: SWC-136
```

## 📝 Description

- The Right-to-Left Override (RTLO) character (U+202E) is a Unicode control character that reverses the rendering direction of following text in editors, terminals, and IDEs. 
- In smart contract development, it can be exploited to:
- Obfuscate malicious filenames (e.g., exploit{U+202E}fdp.js appears as exploitsj.pdf).
- Mask variable or function names in Solidity code.
- Mislead human auditors, reviewers, or automated scanners.
- The vulnerability is not due to a Solidity compiler bug, but due to the obfuscation of intent during manual review or filename-based script auditing.

## 🚨 Vulnerable Code

```solidity
// File appears to be SafeVault.sol, but is actually Malicious⠠fdp.sol due to U+202E
pragma solidity ^0.8.0;

contract LegitContract {
    function withdraw() public {
        // hidden backdoor function via name obfuscation
    }
}
```

## 🧪 Exploit Scenario

1. Attacker creates a Solidity contract named Withdraw\u202Efdp.sol, which renders as Withdraw.pdf.sol.
2. A developer or auditor opens the file and assumes it's a harmless documentation stub.
3. The file actually includes compiled code with backdoors or privileged logic.
4. Deployed contract executes unexpected or harmful actions, bypassing manual review or version control policies.

**Assumptions:**

- Reviewers use visual editors, web interfaces, or command-line tools that fail to sanitize Unicode rendering.
- CI/CD pipelines or security tools rely on filename-based allowlisting or scanning heuristics.

## ✅ Fixed Code

```solidity
function guess(uint n) public payable {
    require(msg.value == 1 ether);
    uint p = address(this).balance;
    checkAndTransferPrize(n, p, msg.sender);
}
```

## 🛡️ Prevention

### Primary Defenses

- Implement a CI rule or linter that rejects files containing RTLO (\u202E), LRO (\u202D), or other bidirectional Unicode markers.
- Strip non-printable or control characters from filenames and identifiers before accepting PRs.

### Additional Safeguards

- Enable editor/IDE plugins that highlight invisible characters.
- Avoid relying on visual appearance of names; enforce code style guidelines.
- Use pre-deployment diff checks and trusted source auditing.

### Detection Methods

- Scan all source files for Unicode control characters.
- Use regular expressions or byte-level search for \u202E, \u202D, \u200F, etc.
- Tools: grep -P '\x{202E}', Slither plugin, ESLint custom rules, Git hooks

## 🕰️ Historical Exploits

- **Name:** Visual Studio RTLO Trojan Files 
- **Date:** 2020 
- **Loss:** Code execution via disguised file extensions 
- **Post-mortem:** [Link to post-mortem](https://cwe.mitre.org/data/definitions/779.html) 
  
## 📚 Further Reading

- [SWC-136: Unencrypted Sensitive Data (related visual confusion)](https://swcregistry.io/docs/SWC-136/)  
- [Trojan Source Attack (Cambridge paper)](https://trojansource.codes/) 
- [GitHub Unicode Security Tools](https://github.com/github/semantic/issues/57) 

---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Right-to-Left Override (RTLO) Character Obfuscation Enables Malicious File or Variable Masking
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 3   
complexity: 3     
detectability: 2  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Misleads reviewers into trusting or ignoring malicious files; can cause loss via disguised behavior.
- **Exploitability**: Trivial to embed RTLO character using a text editor or script.
- **Reachability**: Any source file, repo, or variable can carry this without restrictions.
- **Complexity**: Moderate understanding of Unicode, but easy to automate.
- **Detectability**: Difficult to notice without specialized tools or visual hints.







