# Missing Events for Non-Critical Actions

```YAML
id: TBA
title: Missing Events for Non-Critical Actions 
severity: L
category: observability
language: solidity
blockchain: [ethereum]
impact: Loss of traceability, reduced monitoring and off-chain indexing capabilities
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-778
swc: SWC-136
```

## 📝 Description
- Missing events for non-critical actions occurs when important user or protocol-triggered state changes—such as configuration updates, parameter changes, non-fund movements, or permission grants—are executed without emitting logs. 
- Although this doesn’t directly lead to financial loss or logic bugs, it results in:
- Poor transparency for off-chain indexers and monitoring tools,
- Reduced auditability and debugging difficulty for protocol behavior,
- User experience degradation (e.g., no confirmation for setting preferences, approvals, etc.).
- This issue is common in DAO config changes, permission updates, or protocol settings where changes happen silently.

## 🚨 Vulnerable Code

```solidity
contract Configurable {
    uint256 public fee;

    function setFee(uint256 _fee) external {
        require(_fee <= 100, "Too high");
        fee = _fee;
        // ❌ No event emitted
    }
}
```

## 🧪 Exploit Scenario

Step-by-step impact:

1. Protocol operator updates configuration parameter (e.g., fee, timeout, roles).
2. Off-chain indexers or dashboards have no visibility into the change.
3. Users or devs are unaware of critical updates during debugging or analysis.
4. Trust, usability, and analytics pipelines are affected.

**Assumptions:**

- State changes are valid but unlogged.
- Off-chain services or audits rely on full observability of contract behavior.

## ✅ Fixed Code

```solidity

contract Configurable {
    uint256 public fee;

    event FeeUpdated(uint256 newFee);

    function setFee(uint256 _fee) external {
        require(_fee <= 100, "Too high");
        fee = _fee;
        emit FeeUpdated(_fee); // ✅ Now observable
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Emit events for all state-changing functions, even if non-critical.
- Include:
- Configuration changes
- Access control assignments
- Initialization and update actions

### Additional Safeguards

- Document emitted events in interface/ABI for indexers.
- Use consistent event naming across modules (XUpdated, YGranted, ZRemoved).
- Integrate logs into off-chain dashboards (e.g., The Graph, Dune, Tenderly).

### Detection Methods

- Slither: missing-events, unlogged-state-change, silent-updates detectors.
- Manual code review for functions that modify storage without emit statements.
- Testing off-chain apps to ensure completeness of on-chain logs.

## 🕰️ Historical Exploits

- **Name:** Wizards & Dragons Admin Event Omission 
- **Date:** 2021 
- **Loss:** N/A 
- **Post-mortem:** [Link to post-mortem](https://blog.solidityscan.com/understanding-events-in-smart-contracts-26e8d50b3eef) 

## 📚 Further Reading

- [SWC-136: Unchecked Return Values / Unexpected Behavior](https://swcregistry.io/docs/SWC-136) 
- [Solidity Docs – Events](https://docs.soliditylang.org/en/latest/contracts.html#events) 
- [The Graph – Event-Based Indexing](https://thegraph.com/docs/en/) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Missing Events for Non-Critical Actions 
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

- **Impact**: Limits monitoring and user feedback but doesn’t affect funds or logic directly.
- **Exploitability**: Not exploitable, but creates operational blind spots.
- **Reachability**: Very common across protocol settings and permissioned functions.
- **Complexity**: Simple dev oversight.
- **Detectability**: High — easily flagged by Slither or manual reviews.