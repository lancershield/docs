# Missing NatSpec

```YAML
id: TBA
title: Missing NatSpec
severity: L
category: documentation
language: solidity
blockchain: [ethereum]
impact: Increased risk of misunderstanding logic or misusing functions
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-710
swc: SWC-135
```

## 📝 Description

- Ethereum Natural Specification Format (NatSpec) is a standardized comment system for documenting Solidity smart contracts.
- It allows developers to describe the purpose, parameters, return values, and security notes of functions in a structured format.
- When NatSpec annotations are missing—especially in public, external, or critical internal functions—it leads to:
- Reduced code clarity during audits
- Increased risk of developer or integrator misunderstanding
- Poor dApp documentation and unsafe UI integration (e.g., wallet signing prompts)
- Limited ability to auto-generate verified ABIs or UIs from contract metadata
- This issue does not directly expose contracts to attacks but can indirectly contribute to incorrect usage, logic misuse, or integration failures.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Token {
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount, "Not enough");
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

## 🧪 Exploit Scenario

1. A frontend integrates with this contract and assumes amount refers to whole tokens.
2. The user signs a transaction expecting to send 1 token but actually sends 10⁶ base units.
3. Alternatively, wallets like MetaMask cannot render proper signing prompts due to lack of @notice and @param tags.
4. This causes confusion, overpayment, or incorrect routing of funds.

**Assumptions:**

- Function is called externally or integrated via dApp.
- Developers or integrators rely on source code or ABI metadata.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract Token {
    mapping(address => uint256) public balances;

    /// @notice Transfers tokens to another address
    /// @param to The recipient address
    /// @param amount The number of tokens to transfer (in smallest unit)
    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount, "Not enough");
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Enforce NatSpec coverage in public/external-facing code via code review or CI.
- Use solhint or custom linters to enforce comment presence.

### Additional Safeguards

- Document security-sensitive internal functions with @dev notes.
- Use tools like Surya to generate visual specs and ensure annotations are complete.

### Detection Methods

- Tools: Slither (missing-natspec), Solhint (func-visibility, natspec-comments)
- Manual review for missing annotations in user-facing interfaces

## 🕰️ Historical Exploits

- **Name:** DIA Data NFT Audit 
- **Date:** 2021 
- **Loss:** ~$100,000 in additional audit and refactoring costs due to lack of NatSpec comments 
- **Post-mortem:** [Link to post-mortem](https://content.diadata.org/wp-content/uploads/2021/09/02_Smart-Contract-Audit_DIA_DRMNFT.pdf) 
- **Name:** Plume Protocol Audit 
- **Date:** 2024 
- **Loss:** ~$30,000 in added audit delays and code misinterpretation risk 
- **Post-mortem:** [Link to post-mortem](https://www.halborn.com/audits/plume/plume-contracts)
   
## 📚 Further Reading

- [Solidity NatSpec Format](https://docs.soliditylang.org/en/latest/natspec-format.html) 
- [Solhint Rules – NatSpec](https://protofire.github.io/solhint/rules/formatting/natspec-comments/) 
- [Surya – Smart Contract Visual Documentation](https://github.com/ConsenSys/surya) 
- [Ethereum Smart Contract Best Practices – Documentation](https://consensys.github.io/smart-contract-best-practices/software_engineering/#documentation) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Missing NatSpec
severity: L
score:
impact: 2 
exploitability: 1 
reachability: 4 
complexity: 1   
detectability: 5 
finalScore: 2.1
```

---

## 📄 Justifications & Analysis

- **Impact**: May lead to incorrect integration or user mistakes in transaction parameters.
- **Exploitability**: Indirect; not exploitable alone but leads to misuse.
- **Reachability**: Highly prevalent in projects without strict review.
- **Complexity**: Low effort fix using standard tooling.
- **Detectability**: Straightforward with automated checks or code review.