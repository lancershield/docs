# Incomplete Event Logging

```YAML
id: TBA
title: Incomplete Event Logging 
severity: L
category: observability
language: solidity
blockchain: [ethereum]
impact: Incomplete off-chain data tracking, debugging issues, user distrust
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-778
swc: SWC-136
```

## 📝 Description

- Incomplete event logging occurs when events are emitted but do not include all relevant parameters related to the state change. This weakens:
- Off-chain observability (The Graph, Dune, Tenderly, etc.),
- User-facing logs and dashboards
- Security analysis and post-mortems
- Oracles or bots relying on complete emitted data.
- Typical omissions include:
- Not indexing key values (`indexed` missing),
- Failing to log sender/recipient amounts or state changes (e.g., before/after),
- Not logging important administrative actions or context.

## 🚨 Vulnerable Code

```solidity
contract IncompleteLogging {
    event TokensTransferred(address to);

    function transfer(address to, uint256 amount) external {
        // ✅ Transfer logic here
        emit TokensTransferred(to); // ❌ amount and sender not included
    }
}
```
## 🧪 Exploit Scenario

Step-by-step impact:

1. Contract emits partial event with only to address.
2. Off-chain indexers and dashboards are unable to reconstruct transaction history accurately.
3. Investigators/auditors cannot attribute full context in bug reports or attack post-mortems.
4. Reputation and traceability of the protocol are reduced.

**Assumptions:**

- Important context (e.g., value, sender, config) is omitted from events.
- Off-chain systems or monitoring rely on complete event logs.

## ✅ Fixed Code

```solidity

contract CompleteLogging {
    event TokensTransferred(address indexed from, address indexed to, uint256 amount);

    function transfer(address to, uint256 amount) external {
        // ✅ Transfer logic here
        emit TokensTransferred(msg.sender, to, amount);
    }
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: L
  reasoning: "Transparency and operational tooling are degraded, but core assets are not at risk."
- context: "High-frequency DeFi protocol"
  severity: M
  reasoning: "Indexing and subgraph failures can cascade into severe UX or trading bugs."
- context: "Private or internal-only contract"
  severity: I
  reasoning: "No dependency on logs for external observability."
```

## 🛡️ Prevention

### Primary Defenses

- Include all relevant context in events:
- msg.sender, recipient, token ID, amount, config value, etc.
- Use indexed keywords for key search fields (addresses, IDs).
- Emit events for every important state change, not just token movements.

### Additional Safeguards

- Align events with frontend requirements and The Graph schema.
- Include event coverage in unit and integration tests.
- Use audit checklists to confirm log completeness.

### Detection Methods

- Slither: missing-event-parameters, partial-logging, non-indexed-key detectors.
- Manual audit comparing function state changes to corresponding events.
- Review subgraph schema and matching emitted values.

## 🕰️ Historical Exploits

- **Name:** Level Finance Claim Multiple Bug 
- **Date:** 2023-05
- **Loss:** ~$1 million  
- **Post-mortem:** [Link to post-mortem](https://cointelegraph.com/news/level-finance-confirms-1m-exploit-due-to-buggy-smart-contract) 

## 📚 Further Reading

- [SWC-136: Unexpected Behavior from Missing Events](https://swcregistry.io/docs/SWC-136) 
- [Solidity Docs – Event Logging](https://docs.soliditylang.org/en/latest/contracts.html#events) 
- [The Graph Docs – Event Indexing Best Practices](https://thegraph.com/docs/en/developing/defining-a-subgraph/) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Incomplete Event Logging 
severity: L
score:
impact: 2         
exploitability: 0 
reachability: 5   
complexity: 1     
detectability: 5  
finalScore: 2.1
```

---

## 📄 Justifications & Analysis

- **Impact**: While not financially damaging directly, it affects protocol transparency and reliability.
- **Exploitability**: None directly, but may mask other vulnerabilities.
- **Reachability**: Found in many contracts that use events carelessly or minimally.
- **Complexity**: Very low — results from human error or neglect.
- **Detectability**: High — audit tools and event schema mismatches catch it easily.
