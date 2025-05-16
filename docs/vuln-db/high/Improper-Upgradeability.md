# Improper Upgradeability

```YAML
id: TBA
title: Improper Upgradeability Leading to Storage Corruption 
severity: H
category: upgradeability
language: solidity
blockchain: [ethereum]
impact: Permanent contract malfunction or unauthorized control
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.5.0", "<0.8.21"]
cwe: CWE-694
swc: SWC-112
```

## 📝 Description

- Improper upgradeability refers to flaws in upgradable smart contract systems (e.g., proxy patterns) where either the storage layout is misaligned between proxy and implementation, or the upgrade logic is left exposed to unauthorized parties.
- This can cause storage collisions, logic corruption, or even complete loss of control over the proxy contract.
- Improper handling of `delegatecall` or `admin` storage slots is especially dangerous in patterns like UUPS and Transparent proxies if not implemented using hardened libraries.

## 🚨 Vulnerable Code

```solidity
// Simplified proxy example with unprotected upgrade
contract Proxy {
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

    function upgradeTo(address _newImpl) external {
        implementation = _newImpl; // ❌ No access control
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker calls upgradeTo() and points the proxy to their malicious implementation.
2. All future calls to the proxy are now delegated to attacker code.
3. Attacker can steal funds, disable the contract, or overwrite critical state.

**Assumptions:**

- Upgrade function is public or not gated with onlyOwner or onlyProxyAdmin.
- Implementation contract has unaligned storage or logic.

## ✅ Fixed Code

```solidity
import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeProxyAdmin is Ownable {
    function upgrade(address proxy, address newImplementation) external onlyOwner {
        TransparentUpgradeableProxy(payable(proxy)).upgradeTo(newImplementation);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use OpenZeppelin Proxy libraries (UUPS, TransparentProxy) with hardened admin logic.
- Lock and restrict upgrade functions (onlyOwner, onlyProxyAdmin).
- Maintain strict storage layout discipline between implementation versions.

### Additional Safeguards

- Use storage gaps in inherited contracts for future-safe upgrades.
- Add on-chain version checks and rollback mechanisms.
- Use test tools like hardhat-upgrades to simulate layout compatibility.

### Detection Methods

- Slither: unprotected-upgrade, delegatecall misuse detectors.
- Manual audit of fallback logic and delegatecall boundaries.
- Bytecode diffing and layout validation between upgrades.

## 🕰️ Historical Exploits

- **Name:** Parity Wallet Library Selfdestruct 
- **Date:** 2017-11-06 
- **Loss:** ~$150M locked 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert-2/) 

## 📚 Further Reading

- [SWC-112: Delegatecall to Untrusted Callee – SWC Registry](https://swcregistry.io/docs/SWC-112/) 
- [OpenZeppelin: Upgradeable Contracts Guide](https://docs.openzeppelin.com/upgrades-plugins/1.x/) 
- [Solidity Docs: Initializer Pattern](https://docs.soliditylang.org/en/latest/contracts.html#constructors) 
 
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Improper Upgradeability
severity: H
score:
impact: 5 
exploitability: 4
reachability: 4  
complexity: 3  
detectability: 3  
finalScore: 4.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Entire protocol can be hijacked, frozen, or misdirected via malicious logic.
- **Exploitability**: Simple if upgrade functions are exposed or poorly controlled.
- **Reachability**: Proxy contracts typically expose fallback and upgrade paths.
- **Complexity**: Moderate; requires constructing a compatible attacker implementation.
- **Detectability**: Easily flagged if unsafe proxy patterns are used, but subtle layout misalignments may escape detection.
