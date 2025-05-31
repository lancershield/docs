# Ambiguous Naming 

```YAML
id: LS07I
title: Ambiguous Naming 
baseSeverity: I
category: code-quality
language: solidity
blockchain: [ethereum]
impact: Misinterpretation of contract behavior
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-676
swc: SWC-131
```
## 📝 Description

- Ambiguous naming occurs when functions, variables, or constants are named in a misleading or unclear manner, potentially leading developers or auditors to incorrect assumptions about their behavior. 
- While not directly exploitable in all cases, this can lead to:
- Misuse of sensitive functions
- Misconfiguration of logic (e.g., fee rates or roles)
- Mistakes during upgrades or integrations
- Exploitable behavior if assumptions override true function
- This issue is especially dangerous when the name implies a capped or static value (e.g., MAX_SUPPLY = 10000) but is not enforced in logic.

## 🚨 Vulnerable Code

```solidity

uint256 public MAX_SUPPLY = 10000; // misleading: no logic enforces this

function mint(uint256 amount) public {
    // logic allows minting beyond MAX_SUPPLY
    _mint(msg.sender, amount);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit:

1. A user reads the contract and sees MAX_SUPPLY = 10000.
2. They assume the mint function enforces this limit.
3. Malicious actor continues to mint tokens beyond this threshold.
4. The token economy becomes unstable or diluted.
5. Protocols or users relying on the assumption of a hard cap are misled.

**Assumptions:**

- Developers or integrators rely on variable names without auditing logic.
- There is no external enforcement or override of values through governance.

## ✅ Fixed Code

```solidity

uint256 public constant MAX_SUPPLY = 10000;

function mint(uint256 amount) public {
    require(totalSupply() + amount <= MAX_SUPPLY, "Exceeds cap");
    _mint(msg.sender, amount);
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "Naming misleads developers and users, leading to logic errors."
- context: "DeFi protocol token with hardcap assumptions"
  severity: H
  reasoning: "Ambiguous constants may break integrations or governance trust."
- context: "Internal-only admin contract"
  severity: L
  reasoning: "Impact is limited as developers control deployment and usage."
```

## 🛡️ Prevention

### Primary Defenses

- Use consistent, descriptive, and verifiable naming conventions.
- Avoid constants without logic enforcement.
- Document intent clearly using NatSpec.

### Additional Safeguards

- Explicit checks in code for invariants implied by variable names.
- Internal naming audits before production deployments.

### Detection Methods

- Manual code review for misleading names
- Linting tools (e.g., Solhint, ESLint with Solidity plugin)
- Static analysis to match naming with logic intent

## 🕰️ Historical Exploits

- **Name:** Rubixi Constructor Vulnerability 
- **Date:** 2016   
- **Loss:** Attacker became contract owner due to misnamed constructor
- **Post-mortem:** [Link to post-mortem](https://blog.ethereum.org/2016/06/19/thinking-smart-contract-security) 
  
## 📚 Further Reading

- [SWC-131: Unused or Misleading State Variables – SWC Registry](https://swcregistry.io/docs/SWC-131/)
- [QuillAudits: Top 10 Common Solidity Issues](https://www.quillaudits.com/blog/web3-security/solidity-issues)  
- [SecureFlag: Use of Dangerous Functionality in Smart Contracts](https://knowledge-base.secureflag.com/vulnerabilities/security_misconfiguration/use_of_dangerous_functionality_smart_contracts.html)  

---

## ✅ Vulnerability Report

```markdown
id: LS07I
title: Ambiguous Naming 
severity: I
score:
impact: 3      
exploitability: 2 
reachability: 3  
complexity: 1   
detectability: 4 
finalScore: 2.75
```

---

## 📄 Justifications & Analysis

- **Impact**: Tokenomics and permission logic can be misunderstood, resulting in protocol misbehavior.
- **Exploitability**: Indirect — depends on misinterpretation by humans or third-party integrations.
- **Reachability**: Public variables and function names visible to all users.
- **Complexity**: Simple issue of perception, not technical sophistication.
- **Detectability**: High — linting and manual review tools are sufficient.

