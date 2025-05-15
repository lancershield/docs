# Unverified Upgrade Implementations

```YAML

id: TBA
title: Unverified Upgrade Implementations 
severity: C
category: upgradeability
language: solidity
blockchain: [ethereum]
impact: Protocol logic may be replaced with malicious or incompatible contracts
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.7.0", "<latest"]
cwe: CWE-841
swc: SWC-112
```

## 📝 Description

- Unverified upgrade implementations** occur in upgradable contracts (e.g. UUPS or Transparent Proxies) when the admin or upgrade mechanism allows setting a new implementation address without validating:
- The implementation’s code hash,
- Interface compatibility,
- authorization logic.
- This can lead to:
- Malicious logic injection (e.g., draining funds, disabling the protocol),
- Storage layout mismatch, leading to undefined behavior or fund loss,
- Takeover of proxy contracts via fake `authorizeUpgrade()` or self-destruct logic.

## 🚨 Vulnerable Code

```solidity
contract ProxyAdmin {
    function upgrade(address proxy, address newImplementation) external onlyOwner {
        TransparentUpgradeableProxy(proxy).upgradeTo(newImplementation); // ❌ No validation
    }
}

```

## 🧪 Exploit Scenario

Step-by-step attack:

1. Admin (or attacker with access) upgrades a proxy to a malicious contract.
2. The malicious implementation includes logic like:
selfdestruct(),transferAllTokensTo(attacker),authorizeUpgrade() that accepts any caller.
3. Users interacting with the proxy now unknowingly call into the malicious contract.
4. Funds are stolen, or the contract becomes permanently bricked.

**Assumptions:**

- A legitimate developer deploys an implementation with a mismatched storage layout, accidentally corrupting all state variables (e.g., turning balances into addresses).

##✅ Fixed Code

```solidity

contract SecureProxyAdmin is Ownable {
    mapping(bytes32 => bool) public approvedImplementations;

    function approveImplementation(address impl) external onlyOwner {
        bytes32 hash = keccak256(impl.code);
        approvedImplementations[hash] = true;
    }

    function upgrade(address proxy, address newImplementation) external onlyOwner {
        require(approvedImplementations[keccak256(newImplementation.code)], "Unapproved implementation");
        TransparentUpgradeableProxy(proxy).upgradeTo(newImplementation); // ✅ Safe
    }
}
```


## 🛡️ Prevention

### Primary Defenses

- Maintain an allowlist of approved implementation hashes and validate on upgrade.
- Use OpenZeppelin's Upgrades Plugins, which enforce:
- Storage compatibility,Abstract Upgrade authorization,Implementation transparency.

### Additional Safeguards

- Add upgrade event logs with implementationHash, timestamp, and initiator.
- Protect _authorizeUpgrade() with strict RBAC or governance only.
- Prohibit upgrades if a timelock or DAO governance process is not complete.

### Detection Methods

- Slither: unverified-upgrade, unsafe-upgrade-call, authorizeUpgrade-open.
- Manual audits of upgradeTo() and upgradeToAndCall() functions.
- Simulation tools to test logic and storage layout compatibility.

## 🕰️ Historical Incidents

- **Name:** Audius Upgrade Takeover 
- **Date:** 2022 
- **Impact:** ~$6M stolen by attacker upgrading proxy to malicious logic 
- **Post-mortem:** [https://rekt.news/audius-rekt](https://rekt.news/audius-rekt) 


## 📚 Further Reading

- [SWC-112: Delegatecall to Untrusted Contract](https://swcregistry.io/docs/SWC-112) 
- [OpenZeppelin Upgrade Plugins (Hardhat)](https://docs.openzeppelin.com/upgrades-plugins) 
- [Solidity Upgrade Patterns](https://docs.soliditylang.org/en/latest/contracts.html#contract-upgradeability) 
- [Slither Upgrade Safety Rules](https://github.com/crytic/slither) 


---
## ✅ Vulnerability Report

```markdown
id: TBA
title: Unverified Upgrade Implementations 
severity: C
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 4.4

```



---

## 📄 Justifications & Analysis

- **Impact**: Critical — entire contract behavior and ownership can change.
- **Exploitability**: High if upgrade access is compromised or poorly secured.
- **Reachability**: Common in projects using Transparent/UUPS proxies without validation.
- **Complexity**: Simple to misuse, secure implementation requires good patterns.
- **Detectability**: High — audit and tooling (e.g., OpenZeppelin plugins) can prevent it.