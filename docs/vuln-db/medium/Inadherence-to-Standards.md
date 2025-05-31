# Inadherence to Standards

```YAML
id: LS40M
title: Inadherence to Standards
baseSeverity: M
category: interoperability
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Ecosystem incompatibility, broken integrations, loss of utility
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-749
swc: SWC-112
```

## 📝 Description

- Inadherence to Standards refers to smart contracts that claim to implement widely used interfaces (e.g., ERC20, ERC721, ERC4626) but fail to follow the specifications exactly, leading to:
- Breakage in DeFi integrations
- Wallets and DApps misinterpreting balance, transfer, or approval logic
- Loss of funds due to misaligned expectations between interfaces
- Such bugs are dangerous because they silently affect cross-protocol interactions, often bypassing security audits if tests assume standard behavior.

## 🚨 Vulnerable Code (ERC20 Return Omission)

```solidity

function transfer(address to, uint256 amount) public {
    balances[msg.sender] -= amount;
    balances[to] += amount;
    // ❌ missing `return true`
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A new ERC20 token is launched claiming to follow the ERC20 standard.
2. The token omits return true in its transfer() and transferFrom() methods.
3. A lending protocol (e.g., Compound fork) integrates the token.
4. The protocol's logic checks require(token.transfer(...)), expecting a boolean return.

**Assumptions:**

- The smart contract advertises compatibility with a known standard (e.g., ERC20, ERC721) but violates one or more expected behaviors.
- External integrations (lending protocols, DEXes, wallets) rely on interface-level assumptions and do not validate runtime behavior.

## ✅ Fixed Code

```solidity

function transfer(address to, uint256 amount) public returns (bool) {
    balances[msg.sender] -= amount;
    balances[to] += amount;
    emit Transfer(msg.sender, to, amount);
    return true;
}
```

## 🧭 Contextual Severity

```yaml
- context: "Token or protocol fails standard return values, affecting integrations"
  severity: M
  reasoning: "Leads to silent failures, funds stuck, or broken integrations"
- context: "Used internally only, not exposed to ecosystem"
  severity: L
  reasoning: "Minimal risk if scope is isolated"
- context: "Verified standards-compliant code used"
  severity: I
  reasoning: "No standard mismatch"
```

## 🛡️ Prevention

### Primary Defenses

- Strictly follow token or protocol standards (ERC20, ERC721, ERC4626, etc.)
- Use well-audited base contracts (e.g., OpenZeppelin)
- Include conformance tests using ERC test suites or protocol simulators

### Additional Safeguards

- Validate expected behavior with integration partners (e.g., dApps, bridges)
- Add interface-based unit tests (IERC20.transfer(...) == true)
- Use interface compliance checkers in CI pipelines

### Detection Methods

- Run test suites from official EIP reference implementations
- Tools: Slither (erc20-interface, incorrect-erc20), MythX, OpenZeppelin test harnesses

## 🕰️ Historical Exploits
 
- **Name:** Swerve Finance Token Bug 
- **Date:** 2020-09 
- **Loss:** ~$1,500,000 
- **Post-mortem:** [Link to post-mortem](https://discord.gg/swerve)  
  
📚 Further Reading

- [SWC-135: Incorrect Implementation](https://swcregistry.io/docs/SWC-135) 
- [CWE-693: Protection Mechanism Failure](https://cwe.mitre.org/data/definitions/693.html) 
- [OpenZeppelin ERC20 Docs](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20) 
- [EIP-20 Standard](https://eips.ethereum.org/EIPS/eip-20) 

---

## ✅ Vulnerability Report

```markdown
id: LS40M
title: Inadherence to Standards
severity: M
score:
impact: 4 
exploitability: 3  
reachability: 4  
complexity: 2  
detectability: 4  
finalScore: 3.65
```

---

## 📄 Justifications & Analysis

- **Impact**: Causes systemic failure across protocols relying on standard behavior.
- **Exploitability**: Easily triggered by deploying misbehaving tokens or contracts.
- **Reachability**: Impacts token transfers, approvals, and protocol flows.
- **Complexity**: Low barrier to error but high damage potential.
- **Detectability**: Detectable with compliance tests and static analyzers.
