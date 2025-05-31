# Debug Statements Left

```YAML
id: LS14I
title: Debug Statements Left
baseSeverity: I
category: code-quality
language: solidity
blockchain: [ethereum]
impact: Accidental disclosure or gas waste
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.6.0", "<0.8.24"]
cwe: CWE-489
swc: SWC-135
```

## 📝 Description

- Leaving debug statements such as console.log, emit Debug(...), or internal markers in production smart contracts introduces unnecessary gas overhead and can inadvertently leak sensitive state variables.
- In production environments, these debug artifacts offer no functional benefit and clutter the execution trace, consuming gas and potentially revealing internal logic or variable states that attackers can use to inform exploit paths.

## 🚨 Vulnerable Code

```solidity

import "hardhat/console.sol";

contract Vault {
    uint public balance;

    function deposit() public payable {
        balance += msg.value;
        console.log("Deposit received: ", msg.value); // Debug left in code
    }
}
```

## 🧪 Exploit Scenario

1. Developer tests contract using Hardhat with console.log statements.
2. Debug statement is accidentally left before deployment.
3. When compiled for a non-local network, deployment fails, or gas is wasted unnecessarily.
4. If instead it's a custom emit Debug(...), it bloats logs with redundant data and may leak internal values like keys, addresses, or balances.

**Assumptions** 

- Assumes devs don’t clean up pre-deployment. Public contract visibility allows anyone to read emitted debug data.

## ✅ Fixed Code

```solidity

// Removed console.log for cleaner, production-ready code
contract Vault {
    uint public balance;

    function deposit() public payable {
        balance += msg.value;
    }
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: I
  reasoning: "Adds unnecessary gas usage or log clutter; not a security bug."
- context: "Production Contract with emit-based Debug Events"
  severity: M
  reasoning: "May leak sensitive variables and pollute logs, increasing attack surface."
- context: "Local Test Contract"
  severity: I
  reasoning: "No impact if used in test-only builds or scripts."
```

## 🛡️ Prevention

### Primary Defenses

- Remove all console.log or emit Debug(...) before deploying.
- Use network-aware flags to strip debug-only code during build.

### Additional Safeguards

- Use separate test/development branches or folders.
- Automate lint checks to reject builds containing debug artifacts.

### Detection Methods

- Solhint with custom rules
- Slither can detect event emissions
  
## 🕰️ Historical Exploits

- **Name:** Voltage Finance Reentrancy Exploit
- **Date:** March 2022 
- **Loss:** Approximately $4.67 million 
- **Post-mortem:** [Link to post-mortem](https://cointelegraph.com/news/hacker-from-2022-voltage-finance-exploit-moves-eth-to-tornado-cash)
  
## 📚 Further Reading

- [SWC-135: Code With No Effects – SWC Registry](https://swcregistry.io/docs/SWC-135/)
- [Hardhat Docs: console.log Usage](https://hardhat.org/hardhat-network/reference/#console-log)
- [Slither: Detect Debug Artifacts](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: LS14I
title: Debug Statements Left
severity: I
score:
impact: 1
exploitability: 1
reachability: 5
complexity: 1
detectability: 5
finalScore: 2.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Minimal impact, mostly around gas cost or log pollution.
- **Exploitability**: Requires accidental inclusion and user interaction.
- **Reachability**: Easy to reach in functions containing debug code.
- **Complexity**: Simple mistake; attacker requires no effort.
- **Detectability**: Easily caught via code review or automated tools.