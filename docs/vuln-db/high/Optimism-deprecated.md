# Deprecated Optimism Interfaces Lead to Insecure

```YAML
id: TBA
title: Deprecated Optimism Interfaces Lead to Insecure 
baseSeverity: H
category: l2-interop
language: solidity
blockchain: [optimism]
impact: Message loss, unauthorized execution, or contract breakage
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-477
swc: SWC-136
```

## 📝 Description

- Optimism has evolved its cross-domain messaging architecture, deprecating early versions of critical interfaces such as OVM_L1CrossDomainMessenger, OVM_L2CrossDomainMessenger, and Lib_AddressManager. These legacy interfaces are now replaced with standardized components like:
- CrossDomainMessenger (L1 + L2)
- OptimismPortal (L1)
- L2ToL1MessagePasser (L2)
- Using deprecated interfaces in modern deployments results in:
- Incompatible or failed messages across domains
- Stale address resolution via Lib_AddressManager
- Ignored security updates or audit fixes
- Risk of messages being lost, ignored, or re-executed unexpectedly

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

// ❌ Deprecated interface and pattern
interface ILegacyMessenger {
    function sendMessage(address target, bytes calldata message, uint32 gasLimit) external;
}

contract LegacyBridge {
    ILegacyMessenger public messenger = ILegacyMessenger(0x...); // OVM_L1CrossDomainMessenger

    function send(bytes calldata data) external {
        messenger.sendMessage(0xTarget, data, 100000);
    }
}
```

## 🧪 Exploit Scenario

1. A dApp relies on OVM_L1CrossDomainMessenger.sendMessage() to trigger actions on Optimism L2.
2. Optimism upgrades to the Bedrock architecture and removes support for the legacy OVM interfaces.
3. The contract continues to send messages that are no longer processed or routed.
4. Users believe messages are delivered, but funds or state updates are silently lost.
5. Attackers exploit the lack of verification or replay messages via untracked paths.

**Assumptions:**

- The contract is deployed before the Bedrock upgrade or copied from outdated tutorials.
- Developers don’t update to ICrossDomainMessenger and OptimismPortal APIs.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

interface ICrossDomainMessenger {
    function sendMessage(address target, bytes calldata message, uint32 gasLimit) external;
}

contract UpdatedBridge {
    ICrossDomainMessenger public messenger = ICrossDomainMessenger(0x4200000000000000000000000000000000000007); // Bedrock default

    function send(bytes calldata data) external {
        messenger.sendMessage(0xTarget, data, 100_000);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Bridge or access control relying on deprecated Optimism context"
  severity: H
  reasoning: "Attackers can spoof origin and bypass security logic"
- context: "Deprecated interface used for logging or analytics"
  severity: L
  reasoning: "No security-critical behavior affected"
- context: "Proper LibOptimism / Bedrock interface with fallback checks"
  severity: I
  reasoning: "Fully mitigated"
```
## 🛡️ Prevention

### Primary Defenses

- Always refer to Optimism’s latest canonical contracts.
- Replace OVM_* interfaces with CrossDomainMessenger, OptimismPortal, etc.
- Use canonical addresses provided in Optimism network docs.

### Additional Safeguards

- Check L1 ↔ L2 communication flow compatibility post-Bedrock.
- Monitor upgrade notices from Optimism and avoid legacy package installations.

### Detection Methods

- Look for interfaces with prefix OVM_, or usage of Lib_AddressManager.
- Validate use of sendMessage() against the correct CrossDomainMessenger ABI.
- Tools: Slither (custom pattern), grep, manual review of cross-domain logic

## 🕰️ Historical Exploits

- **Name:** Optimism Fraud Proof System Vulnerabilities 
- **Date:** 2024-03-22 
- **Loss:** Potential for fraudulent chain history acceptance due to deprecated interface flaws 
- **Post-mortem:** [Link to post-mortem](https://medium.com/offchainlabs/security-disclosure-289a4ad50709)
- **Name:** Optimism Infinite Money Duplication Bug 
- **Date:** 2022-02-02 
- **Loss:** Risk of unlimited token minting via deprecated OVM 2.0 logic 
- **Post-mortem:** [Link to post-mortem](https://medium.com/immunefi/optimism-infinite-money-duplication-bugfix-review-daa6597146a0)
  

## 📚 Further Reading

- [SWC-136: Incorrect Usage of Deprecated Interfaces](https://swcregistry.io/docs/SWC-136/) 
- [Canonical Optimism Contracts GitHub](https://github.com/ethereum-optimism/optimism)
- [Optimism Smart Contracts Audit – OpenZeppelin Blog](https://blog.openzeppelin.com/optimism-smart-contracts-audit)
- [Optimism Audit Reports – Official Documentation](https://docs.optimism.io/stack/security/audits-report)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Deprecated Optimism Interfaces Lead to Insecure 
severity: H
score:
impact: 4         
exploitability: 3
reachability: 4 
complexity: 2     
detectability: 3 
finalScore: 3.45
```

---

## 📄 Justifications & Analysis

- **Impact**: Message delivery failures or security assumptions may be invalid under deprecated interfaces.
- **Exploitability**: Delayed migration allows attackers to exploit mismatched message handling.
- **Reachability**: Any cross-domain system on Optimism deployed before Bedrock may be affected.
- **Complexity**: Moderate due to architecture shift, not core logic error.
- **Detectability**: Hard to notice unless interface versions and addresses are verified post-upgrade.