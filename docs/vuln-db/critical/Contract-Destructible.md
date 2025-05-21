# Contract Destructible

```YAML
id: TBA
title: Contract Destructible 
severity: C
category: access-control
language: solidity
blockchain: [ethereum]
impact: Permanent deletion of contract and loss of funds
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-284
swc: SWC-106
```

## 📝 Description

- If a contract contains a selfdestruct() (or the deprecated suicide()) instruction that is not properly restricted, any external account can trigger it, permanently removing the contract from the blockchain and transferring all Ether held by it to a designated address.
- This typically occurs when:
- There is no access control on the selfdestruct function.
- Ownership checks (require(msg.sender == owner)) are missing or broken.
- The contract is mistakenly deployed with test-only destruct logic.
- The destruction is irreversible and removes all code and storage, breaking integrations and potentially locking or misrouting assets.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Killable {
    function destroy() external {
        selfdestruct(payable(msg.sender)); // ❌ No access control
    }

    receive() external payable {}
}
```

## 🧪 Exploit Scenario

1. A developer includes a destroy() function for testing purposes but forgets to remove or restrict it.
2. A malicious actor finds the unprotected function on mainnet.
3. They call destroy() and trigger selfdestruct, sending all contract funds to their wallet.
4. The contract is permanently removed, and users can no longer interact with it.

**Assumptions:**

- No onlyOwner or require check exists in the selfdestruct function.
- The contract holds Ether or is integrated with other protocols.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeKillable is Ownable {
    function destroy() external onlyOwner {
        selfdestruct(payable(owner())); // ✅ Access control enforced
    }

    receive() external payable {}
}
```

## 🛡️ Prevention

### Primary Defenses

- Enforce access control (onlyOwner, require(msg.sender == owner)) on any self-destruct logic.
- Avoid including self-destruct logic in production unless absolutely necessary.

### Additional Safeguards

- Use isKillable flags in dev/test contracts and strip them during deployment.
- Implement proxy pattern instead of destruct/replace patterns for upgradeability.

### Detection Methods

- Look for selfdestruct() or suicide() calls without access control.
- Tools: Slither (unrestricted-selfdestruct), MythX, custom linting rules

## 🕰️ Historical Exploits

- **Name:** Rubixi Selfdestruct Scam 
- **Date:** 2016 
- **Loss:** ~50 ETH 
- **Post-mortem:** [Rubixi Solidity Bug Explained](https://theethereum.wiki/w/index.php/Rubixi) 
  
## 📚 Further Reading

- [SWC-106: Unprotected Selfdestruct](https://swcregistry.io/docs/SWC-106/) 
- [Solidity Docs – selfdestruct](https://docs.soliditylang.org/en/latest/control-structures.html#selfdestruct) 
- [OpenZeppelin Ownable](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable) 
  
---
  
## ✅ Vulnerability Report

```markdown
id: TBA
title: Contract Destructible 
severity: C
score:
impact: 5  
exploitability: 5 
reachability: 5   
complexity: 1   
detectability: 5 
finalScore: 4.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Total loss of contract, funds, and dependent dApps.
- **Exploitability**: Requires no privileges—anyone can trigger the function.
- **Reachability**: Fully exposed via external or public visibility.
- **Complexity**: Trivial mistake; often due to leftover test logic.
- **Detectability**: Easily caught via Slither, MythX, or visual audit.