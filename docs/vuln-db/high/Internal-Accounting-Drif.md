#  Internal Accounting Drift

```YAML
id: LS12H
title: Internal Accounting Drift 
baseSeverity: H
category: accounting
language: solidity
blockchain: [ethereum]
impact: Fund misaccounting, stuck balances, or undeclared liabilities
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-682
swc: SWC-135
```

## 📝 Description

- Internal Accounting Drift occurs when a smart contract maintains internal variables (e.g., `totalSupply`, `totalDeposits`, `userBalances`) that are not consistently synchronized with actual on-chain token or ETH balances. This mismatch can result in:
- Users receiving incorrect rewards or redemptions,
- Contracts holding more or less than expected,
- Critical invariants (e.g., sum of balances = total supply) being violated.Such drift often occurs due to:
- Skipped updates on edge-case paths,
- External token transfers bypassing accounting logic,
- Failed transfer/revert paths not rolling back state updates.

## 🚨 Vulnerable Code

```solidity
contract AccountingDrift {
    mapping(address => uint256) public balances;
    uint256 public totalDeposits;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
        totalDeposits += msg.value;
    }

    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient");
        balances[msg.sender] -= amount;
        totalDeposits -= amount;

        // ❌ If this fails, state is already updated
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Transfer failed");
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. User calls withdraw(10 ether).
2. Contract updates internal state: balances[msg.sender] -= 10, totalDeposits -= 10.
3. External call to msg.sender.call{value: 10 ether} fails due to gas or reentrancy.
4. Funds remain in the contract, but internal accounting thinks they're gone.
5. Over time, internal state and actual balance drift further apart, breaking audits, refunds, or redemption logic.

**Assumptions:**

- No rollback or refund mechanism for failed ETH transfer.
- External token transfers into the contract are not reflected in totalDeposits.

## ✅ Fixed Code

```solidity

function withdraw(uint256 amount) external {
    require(balances[msg.sender] >= amount, "Insufficient");

    // Move effects after successful transfer
    (bool sent, ) = msg.sender.call{value: amount}("");
    require(sent, "Transfer failed");

    balances[msg.sender] -= amount;
    totalDeposits -= amount;
}
```

## 🧭 Contextual Severity

```yaml
- context: "Vault, pool, or staking contract using dual accounting"
  severity: H
  reasoning: "Fund recovery, redemptions, or payouts can break permanently"
- context: "Cosmetic logic (e.g., dashboard or view-only)"
  severity: L
  reasoning: "No financial loss occurs"
- context: "Force-send and token sweep logic explicitly handled"
  severity: I
  reasoning: "Fully mitigated"
```

## 🛡️ Prevention

### Primary Defenses

- Follow the Checks-Effects-Interactions pattern: update state only after successful external calls.
- Use pull-based payment models to eliminate transfer failure impact.
- Regularly reconcile internal state with external reality (e.g., assert(address(this).balance == totalDeposits)).

### Additional Safeguards

- Emit DriftDetected events if drift is discovered by monitoring tools.
- Add pause/emergency flags if internal balance exceeds a threshold delta.
- Consider integrating invariant tests via Echidna or Foundry fuzzing.

### Detection Methods

- Slither: effects-before-interactions, incorrect-accounting, state-drift detectors.
- Manual inspection for mismatched state updates across all execution paths.
- Unit tests simulating revert/failure edge cases and skipped state updates.

## 🕰️ Historical Exploits

- **Name:** Compound Protocol Reward Distribution Bug 
- **Date:** 2021-09-30 
- **Loss:** Approximately $80 million 
- **Post-mortem:** [Link to post-mortem](https://cryptoslate.com/how-the-tiniest-of-errors-resulted-in-an-80-million-loss-for-compound-finance/) 


## 📚 Further Reading

- [SWC-135: Code With No Effects / Broken Accounting](https://swcregistry.io/docs/SWC-135) 
- [Solidity Docs – Checks-Effects-Interactions](https://docs.soliditylang.org/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) 
- [Trail of Bits – How to Track Accounting Drift](https://blog.trailofbits.com/) 

---

## ✅ Vulnerability Report

```markdown
id: LS12H
title: Internal Accounting Drift 
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 3  
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Breaks the financial guarantees and invariant assumptions of the protocol.
- **Exploitability**: Can be leveraged for griefing, DoS, or disproportionate rewards.
- **Reachability**: Present in most financial protocols with accounting logic.
- **Complexity**: Requires knowledge of failure conditions or state drift sources.
- **Detectability**: Not always obvious unless dynamic testing is performed.
