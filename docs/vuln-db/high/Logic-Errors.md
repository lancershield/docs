# Logic Errors in Contract

```YAML
id: TBA
title: Logic Errors in Contract
baseSeverity: H
category: logic
language: solidity
blockchain: [ethereum]
impact: Unexpected or incorrect behavior of core business logic
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<latest"]
cwe: CWE-840
swc: SWC-135
```
## 📝 Description

- Logic errors occur when the contract code executes successfully but the implemented logic does not match the intended behavior.
- These issues are often found in conditionals, arithmetic operations, loop boundaries, state updates, or access control.
- Unlike syntax or runtime errors, logic flaws don’t revert—making them harder to detect but potentially harmful in production.

## 🚨 Vulnerable Code

```solidity
contract RewardVault {
    mapping(address => uint256) public balances;

    function claimReward() external {
        require(balances[msg.sender] > 0, "No reward");
        balances[msg.sender] = 0;
        payable(msg.sender).transfer(balances[msg.sender]); // BUG: transfers zero
    }
}
```

## 🧪 Exploit Scenario

1. Step-by-step exploit process:
2. User has a positive reward balance.
3. Calls claimReward().
4. Balance is reset before transfer → sends 0 ETH.
5. Funds remain trapped in the contract indefinitely.

**Assumptions:**

- The contract sets balances during other user interactions.
- No fallback to manually recover or fix state errors.

## ✅ Fixed Code

```solidity

function claimReward() external {
    uint256 reward = balances[msg.sender];
    require(reward > 0, "No reward");
    balances[msg.sender] = 0;
    payable(msg.sender).transfer(reward);
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Impacts user-facing functions and may cause financial miscalculations."
- context: "DAO-managed vault with reward logic"
  severity: C
  reasoning: "Misdistribution can result in systemic governance failures."
- context: "Internal-only logic with limited user interaction"
  severity: M
  reasoning: "Limited damage but still requires correction."
```

## 🛡️ Prevention

## Primary Defenses

- Carefully order operations (read before write).
- Use clear variable naming and intermediate variables for clarity.
- Test all edge cases, including zero values and boundary conditions.

### Additional Safeguards

- Implement invariant checks and assertions (assert()).
- Write unit + integration tests for every critical business function.
- Perform formal verification for critical logic flows.

### Detection Methods

- Hardhat/Waffle test coverage on all execution paths.
- Slither: unused-return, incorrect-transfer-order, and logic detectors.
- Symbolic execution via tools like Mythril or Manticore.

## 🕰️ Historical Exploits

- **Name:** KiloEx TrustedForwarder Exploit 
- **Date:** April 2025 
- **Loss:** Approximately $7 million 
- **Post-mortem:** [Link to post-mortem](https://thedailysun.co.za/2025/04/21/kiloex-reveals-details-of-7-million-smart-contract-exploit-in-post-mortem-report/) 

## 📚 Further Reading

- [SWC-124: Incorrect Inheritance Order or Logic](https://swcregistry.io/docs/SWC-124)
- [OWASP Smart Contract Top 10: Logic Errors – OWASP Foundation](https://owasp.org/www-project-smart-contract-top-10/2025/en/src/SC03-logic-errors.html) 
- [OpenZeppelin – Avoiding Logic Errors](https://docs.openzeppelin.com/contracts/4.x/api/utils)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Logic Errors in Contract 
severity: H
score:
impact: 3  
exploitability: 3
reachability: 4  
complexity: 2  
detectability: 3  
finalScore: 3.15
```

## 📄 Justifications & Analysis

- **Impact**: Can block fund withdrawals, misassign roles, or result in protocol malfunction.
- **Exploitability**: Triggered by standard user interactions with crafted input or timing.
- **Reachability**: Often in user-facing functions like withdraw(), claim(), or mint().
- **Complexity**: May require understanding of internal state and order of operations.
- **Detectability**: Harder to catch unless tests or formal analysis are thorough.
