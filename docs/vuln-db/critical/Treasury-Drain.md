# Treasury Drain 

```YAML
id: TBA
title: Treasury Drain via Governance Exploit
baseSeverity: C
category: governance-manipulation
language: solidity
blockchain: [ethereum]
impact: Unauthorized treasury depletion
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<0.8.21"]
cwe: CWE-284
swc: SWC-136
```

## 📝 Description

- A Treasury Drain via Governance Exploit occurs when attackers manipulate a protocol’s governance system to pass malicious proposals that transfer protocol funds to attacker-controlled wallets. 
- If token-based voting lacks quorum, delay, or validation mechanisms, attackers with sufficient voting power (or flashloaned tokens) can execute high-impact proposals to siphon funds from the treasury.

## 🚨 Vulnerable Code

```solidity

function execute(uint proposalId) external {
    Proposal storage p = proposals[proposalId];
    require(block.timestamp > p.endTime, "Voting ongoing");
    require(!p.executed, "Already executed");

    if (p.votesFor > p.votesAgainst) {
        (bool success, ) = p.target.call(p.data); // Dangerous unchecked call
        require(success, "Call failed");
        p.executed = true;
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker accumulates or flashloans enough governance tokens.
2. Submits a proposal that executes transfer() or arbitrary code to drain treasury funds.
3. Uses their voting power to pass the proposal.
4. Calls execute() to finalize and trigger the malicious transfer.

**Assumptions:**

- No time-lock or proposal vetting mechanism.
- Token voting is sufficient for passing proposals (no multi-sig or off-chain governance oversight).
- Treasury or admin contracts are callable via arbitrary data in proposals.

## ✅ Fixed Code

```solidity

function execute(uint proposalId) external onlyGovernance {
    Proposal storage p = proposals[proposalId];
    require(block.timestamp > p.endTime + delayPeriod, "Grace period not met");
    require(!p.executed, "Already executed");
    require(p.votesFor > p.votesAgainst, "Not approved");
    require(_isWhitelistedTarget(p.target), "Invalid target");

    (bool success, ) = p.target.call(p.data);
    require(success, "Execution failed");
    p.executed = true;
}
```

## 🧭 Contextual Severity

```yaml
- context: "DAO with direct treasury access via proposal executor"
  severity: C
  reasoning: "Attack enables immediate theft of all funds on proposal execution."
- context: "Governance protected by timelock, multisig, or review delay"
  severity: M
  reasoning: "Exploitability reduced due to execution guardrails."
- context: "Proposal flow decoupled from fund-moving authority"
  severity: L
  reasoning: "Treasury cannot be drained directly; voting triggers approval only."
```

## 🛡️ Prevention

### Primary Defenses

- Introduce time-locks before execution of high-risk proposals.
- Whitelist executable function signatures or target addresses.
- Require multi-signature confirmation for treasury transfers.

### Additional Safeguards

- Enforce quorum and minimum participation thresholds.
- Restrict execution authority to trusted governance modules.
- Monitor and flag large or suspicious fund movements via governance.

## Detection Methods

- Slither rules for arbitrary-call, dangerous delegatecall, or governance risk.
- Static analysis for unrestricted .call() usage inside proposal execution logic.
- Runtime testing of proposal flows with mock malicious proposals.

## 🕰️ Historical Exploits

- **Name:** Beanstalk Protocol Governance Exploit 
- **Date:** 2022-04-17 
- **Loss:** ~$181 million 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/beanstalk-rekt/) 
- **Name:** Tornado Cash Governance Takeover 
- **Date:** 2023-05-20 
- **Loss:** ~$2.77 million 
- **Post-mortem:** [Link to post-mortem](https://neptunemutual.com/blog/understanding-tornado-cash-exploit/) 
  

## 📚 Further Reading

- [SWC-136: Unchecked External Call](https://swcregistry.io/docs/SWC-136/) 
- [Metana: Governance Attacks in Smart Contracts](https://metana.io/blog/governance-attacks-in-smart-contracts/) 
- [OpenZeppelin: Governor Pattern Documentation](https://docs.openzeppelin.com/contracts/4.x/governance) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Treasury Drain via Governance Exploit
severity: C
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 3  
finalScore: 4.2
```

---

## 📄 Justifications & Analysis

- **Impact**: Complete treasury drain or unauthorized reallocation of funds.
- **Exploitability**: Feasible if attacker can accumulate or borrow voting power.
- **Reachability**: Common in protocols with on-chain governance and token-based proposals.
- **Complexity**: Medium; requires some understanding of proposal formatting.
- **Detectability**: Static tools may not catch economic governance risks without human review.
