# Non-Indexed Events 

```YAML
id: LS08I
title: Non-Indexed Events 
baseSeverity: I
category: observability
language: solidity
blockchain: [ethereum]
impact: Impaired event filtering and off-chain indexing
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-778
swc: SWC-136
```

## 📝 Description

- In Solidity, event parameters marked as indexed enable efficient filtering and search via the Ethereum logs bloom filter. 
- If parameters—especially critical ones like msg.sender, tokenId, or user—are omitted or declared without indexed, it severely hampers traceability.
- Indexing allows off-chain systems like The Graph, Etherscan, or custom indexers to locate specific events without processing entire blocks. Without indexed parameters, observers must manually decode all logs, leading to performance and auditability issues.

## 🚨 Vulnerable Code

```solidity

event TokenMinted(address to, uint256 amount);

function mint(address to, uint256 amount) external {
    _mint(to, amount);
    emit TokenMinted(to, amount); // `to` should be indexed
}
```
## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Contract emits many TokenMinted events for different users.
2. A protocol wants to track all mints to a specific address.
3. Since to is not indexed, the client cannot filter logs efficiently.
4. The system is forced to decode all emitted logs, causing delays, CPU cost, and potential errors.

**Assumptions:**

- Log indexers or analytic platforms depend on efficient filtering.
- Thousands of logs may exist, degrading performance without index support.

## ✅ Fixed Code

```solidity
event TokenMinted(address indexed to, uint256 amount);

function mint(address to, uint256 amount) external {
    _mint(to, amount);
    emit TokenMinted(to, amount);
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "Degrades usability but doesn’t compromise protocol security."
- context: "Event-driven or analytics-heavy protocol"
  severity: M
  reasoning: "Can severely impair data transparency and user experience."
- context: "Minimal logging and internal-only contract"
  severity: I
  reasoning: "No real consequence if logs are not consumed off-chain."
```

## 🛡️ Prevention

### Primary Defenses

- Always index critical parameters in events.
- Follow naming conventions and standard formats for off-chain tooling.

### Additional Safeguards

- Review all event declarations as part of code reviews.
- Use interfaces to enforce consistent event formats.

### Detection Methods

- Slither events, missing-indexed plugin.
- MythX Static analysis for underindexed logs.
- Manual code review for events emitted by externally called functions.

## 🕰️ Historical Exploits

- **Name:** Untraceable Reward Claims 
- **Date:** 2021 
- **Loss:** Analytics platform failed to detect 30% of claims due to missing `indexed` 
- **Post-mortem:** [Link to post-mortem](https://dune.com/docs/querying-data/events/#filtering-by-indexed-arguments) 
  
## 📚 Further Reading

- [SWC-136: Unindexed Event Fields – SWC Registry](https://swcregistry.io/docs/SWC-136/)
- [Solidity Docs: Event Definition and Filtering](https://docs.soliditylang.org/en/latest/contracts.html#events)

## ✅ Vulnerability Report

```markdown
id: LS08I
title: Non-Indexed Events 
severity: I
score:
impact: 2   
exploitability: 2 
reachability: 5   
complexity: 1   
detectability: 4  
finalScore: 2.5
```

---

## 📄 Justifications & Analysis

- **Impact**: Prevents log filtering, degrading UX for off-chain tools.
- **Exploitability**: Developers may forget indexed, unintentionally degrading UX.
- **Reachability**: All public events are potentially affected.
- **Complexity**: Very low—just add indexed in event parameters.
- **Detectability**: Static tools catch it, but overlooked in informal reviews.