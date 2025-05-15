# Logic Errors in Contract Conditions

```YAML

id: TBA
title: Logic Errors in Contract Conditions or Calculations
severity: M
category: logic-error
language: solidity
blockchain: [ethereum]
impact: Unintended behavior or incorrect fund/state handling
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-840
swc: SWC-124
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

- **Name:** Rubixi Early Ownership Bug
- **Date:** 2016-05-03
- **Loss:** N/A
- **Post-mortem:** [Link to post-mortem](https://medium.com/@PracticalDevthe-rubixi-bug-a-smart-contract-vulnerability-with-an-interesting-history-c06c41f5a6b8)
- **Name:** YAM Finance Overflow Bug
- **Date:** 2020-08-13
- **Loss:** Protocol disabled, $750K lost
- **Post-mortem:** [Link to post-mortem](https://medium.com/yam-finance/yam-post-mortem-39467ab8971f)

## 📚 Further Reading

- [SWC-124: Incorrect Inheritance Order or Logic](https://swcregistry.io/docs/SWC-124)
- [Trail of Bits: Logic Bugs in Smart Contracts](https://blog.trailofbits.com/2022/10/how-to-break-smart-contracts/)
- [OpenZeppelin – Avoiding Logic Errors](https://docs.openzeppelin.com/contracts/4.x/api/utils)

## ✅ Vulnerability Report

```markdown
id: vuln\_\_005
title: Logic Errors in Contract Conditions or Calculations
severity: M
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
