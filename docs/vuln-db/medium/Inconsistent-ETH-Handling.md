# Inconsistent ETH Handling

```YAML
id: TBA
title: Inconsistent ETH Handling
baseSeverity: M
category: ether-transfer
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Inadvertent ETH loss, unclaimable funds, or logic bypass
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-703
swc: SWC-113
```

## 📝 Description

- Proxy contracts often forward function calls and storage to an implementation contract via delegatecall. However, if ETH handling (i.e., msg.value) is not carefully managed, especially when using low-level delegatecall, this can lead to:
- Transactions that revert unexpectedly due to missing payable modifiers,
- ETH being sent to a contract that doesn’t receive it, causing loss,
- Inconsistencies in behavior between the proxy and implementation,
- ETH stuck in the proxy with no mechanism to withdraw.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Proxy {
    address public implementation;

    constructor(address _impl) {
        implementation = _impl;
    }

    fallback() external payable {
        (bool success, ) = implementation.delegatecall(msg.data); // ❌ msg.value is ignored by delegatecall target unless explicitly handled
        require(success, "Delegatecall failed");
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A developer deploys a proxy contract with a payable fallback() and sets an implementation contract that includes a logic function like deposit().
2. A user sends a transaction to the proxy with msg.value > 0, expecting ETH to reach the logic’s deposit() function.
3. However, the implementation function is not marked payable, or msg.value is ignored due to improper forwarding.

**Assumptions:**

- Proxy uses raw delegatecall without enforcing payable enforcement or value management.
- Implementation is not built to handle native ETH transfers.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeProxy {
    address public implementation;

    constructor(address _impl) {
        implementation = _impl;
    }

    fallback() external payable {
        _delegate(implementation);
    }

    receive() external payable {}

    function _delegate(address _impl) internal {
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), _impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
```
## 🧭 Contextual Severity

```yaml
- context: "DeFi protocol with refund, claim, or treasury logic tied to raw ETH balance"
  severity: M
  reasoning: "Leads to unclaimable funds or logic breakage"
- context: "Contract has strict internal accounting and rejects unsolicited ETH"
  severity: L
  reasoning: "Risk significantly mitigated"
- context: "Non-payable contract with no ETH handling features"
  severity: I
  reasoning: "Vulnerability irrelevant"
```

## 🛡️ Prevention

### Primary Defenses

- Design proxy/implementation pairs to both handle ETH coherently.
- Mark fallback and delegate targets as payable if ETH is expected.
- Implement tests for ETH transfers via proxy paths.

### Additional Safeguards

- Emit an ETHReceived event on proxy to trace all inbound ETH.
- Consider forwarding all ETH explicitly to implementation, if safe.
- Log a warning or revert if ETH is received but not handled.

### Detection Methods

- Static analysis of fallback function with msg.value > 0 but no payable in implementation.
- Tools: Slither (missing-payable, unhandled-value), Echidna fuzzing

## 🕰️ Historical Exploits

- **Name:** EIP-1967 Proxy ETH Trap 
- **Date:** 2020 
- **Loss:** N/A (design issue caused ETH to be stuck in the proxy with no withdrawal path) 
- **Post-mortem:** [Link to post-mortem](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#delegatecall-and-inheritance) 
  
## 📚 Further Reading

- [SWC-104: Unchecked Call Return Value](https://swcregistry.io/docs/SWC-104) 
- [CWE-703: Improper Check or Handling of Exceptional Conditions](https://cwe.mitre.org/data/definitions/703.html)  
- [Solidity Docs – Fallback and Receive](https://docs.soliditylang.org/en/latest/contracts.html#fallback-function) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Inconsistent ETH Handling 
severity: M
score:
impact: 4      
exploitability: 3 
reachability: 4   
complexity: 3   
detectability: 4 
finalScore: 3.75
```

---

## 📄 Justifications & Analysis

- **Impact**: ETH may be permanently locked or critical flows may revert
- **Exploitability**: Requires user interaction, but attacker may craft confusion
- **Reachability**: Present in nearly every upgradeable proxy architecture
- **Complexity**: Arises from subtle differences in call, delegatecall, and value semantics
- **Detectability**: Easy to catch via test or audit once value routing is traced