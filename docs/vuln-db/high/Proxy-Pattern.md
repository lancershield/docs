# Proxy Pattern Vulnerabilities

```YAML
id: TBA
title: Proxy Pattern Vulnerabilities 
severity: H
category: upgradeability
language: solidity
blockchain: [ethereum]
impact: Logic corruption, upgrade hijacking, or permanent DoS
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.5.0", "<0.8.21"]
cwe: CWE-284
swc: SWC-112
```

## 📝 Description

- Proxy pattern vulnerabilities arise when smart contracts use a delegatecall-based proxy architecture (e.g., UUPS, Transparent Proxy) and:
- Misconfigure the implementation address
- Fail to restrict upgrade/admin functions,
- Allow storage layout collisions between proxy and logic contracts,
- Permit selfdestruct in the logic contract.
- Such issues can result in:
- Total logic override by unauthorized actors,
- Destruction of the proxy contract,
- Permanent lock-up or corruption of state.

## 🚨 Vulnerable Code

```solidity
contract InsecureProxy {
    address public implementation;

    fallback() external payable {
        address impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }

    // ❌ No access control
    function upgradeTo(address newImpl) external {
        implementation = newImpl;
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Proxy contract is deployed and logic implementation is set.
2. upgradeTo() is public and unprotected.
3. Attacker calls upgradeTo() and sets the implementation to a malicious contract.
4. Future calls through proxy are now routed to attacker logic, allowing fund drain, DoS, or selfdestruct.

**Assumptions:**

- No onlyOwner or access control on upgradeTo().
- Proxy storage overlaps with logic contract’s layout.

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
import "@openzeppelin/contracts/proxy/transparent/ProxyAdmin.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureAdmin is Ownable {
    ProxyAdmin public proxyAdmin;

    constructor(address _proxyAdmin) {
        proxyAdmin = ProxyAdmin(_proxyAdmin);
    }

    function upgrade(address proxy, address newImpl) external onlyOwner {
        proxyAdmin.upgrade(TransparentUpgradeableProxy(payable(proxy)), newImpl);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use OpenZeppelin’s Transparent Proxy or UUPS Proxy with ProxyAdmin.
- Enforce strict onlyOwner or onlyProxyAdmin modifiers on all upgrade-related functions.
- Always validate that new implementation addresses do not include selfdestruct logic.

### Additional Safeguards

- Maintain a strict storage layout discipline across implementation versions.
- Use storage gaps in inherited contracts for upgrade safety.
- Add on-chain upgrade logs or require governance approval.

### Detection Methods

- Slither: unprotected-upgrade, delegatecall-in-fallback, admin-access detectors.
- Manual inspection of fallback functions, upgrade methods, and delegatecall usage.
- Compare storage layouts of proxy and implementation to detect collisions.

## 🕰️ Historical Exploits

- **Name:** Parity Multisig Wallet Destruct Bug 
- **Date:** 2017-11-06 
- **Loss:** ~$150M frozen 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert-2/) 

## 📚 Further Reading

- [SWC-112: Delegatecall to Untrusted Callee](https://swcregistry.io/docs/SWC-112) 
- [OpenZeppelin Proxy Pattern Guide](https://docs.openzeppelin.com/upgrades-plugins/1.x/) 
- [Solidity Docs – delegatecall Caveats](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#delegatecall-callcode-and-libraries) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Proxy Pattern Vulnerabilities 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 4  
finalScore: 4.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Attacker can hijack proxy logic or selfdestruct implementation.
- **Exploitability**: High if access control is weak or initialization missing.
- **Reachability**: Common in systems using upgradable patterns.
- **Complexity**: Requires minimal setup—just a crafted implementation contract.
- **Detectability**: Detectable with static analysis and manual audits, especially in fallback paths.
