# Improperly Initialized UUPS Proxy 

```YAML
id: TBA
title: Improperly Initialized UUPS Proxy 
baseSeverity: C
category: upgradeability
language: solidity
blockchain: [ethereum]
impact: Proxy may be permanently bricked or hijacked by unauthorized parties
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.7.0", "<latest"]
cwe: CWE-665
swc: SWC-118
```

## 📝 Description

- Improperly initialized UUPS proxies occur when the implementation contract is deployed without an initialization guard, allowing anyone to call `initialize()` and permanently take over ownership or set critical variables. 
- In UUPS (Universal Upgradeable Proxy Standard), the logic contract itself contains upgrade and ownership logic. If `initialize()` is not protected:
- Attackers can hijack the implementation, upgrade it, or set themselves as the admin.
- Or, the proxy may reuse the same logic, and multiple instances of `initialize()` may brick the proxy by overwriting its state incorrectly.
- This has caused multiple high-profile takeovers and fund freezes in DeFi and NFT platforms using OpenZeppelin UUPS.

## 🚨 Vulnerable Code

```solidity
contract UUPSLogic is UUPSUpgradeable, OwnableUpgradeable {
    function initialize(address _owner) public {
        __Ownable_init();
        transferOwnership(_owner); // ❌ No initializer modifier
    }

    function _authorizeUpgrade(address) internal override onlyOwner {}
}
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. UUPS implementation is deployed without initializer modifier.
2. Attacker calls initialize() and sets themselves as the owner.
3. Now the attacker can call upgradeTo() and deploy a malicious logic contract.
4. Proxy contracts reusing this logic become fully compromised or bricked.

**Assumptions:**

- The implementation contract is publicly deployed and exposed.
- initialize() is callable more than once or by any user.

## ✅ Fixed Code

```solidity

contract UUPSLogic is UUPSUpgradeable, OwnableUpgradeable {
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers(); // ✅ Prevents `initialize()` from being called directly
    }

    function initialize(address _owner) public initializer {
        __Ownable_init();
        transferOwnership(_owner);
    }

    function _authorizeUpgrade(address) internal override onlyOwner {}
}
```

## 🧭 Contextual Severity

```yaml
- context: "Mainnet deployed UUPS proxy without initialize() call"
  severity: C
  reasoning: "Allows full contract takeover and fund loss within one transaction."
- context: "Proxy uses `onlyProxy` + constructor disables initializer"
  severity: L
  reasoning: "Initialization protected; no takeover possible."
- context: "Development/test deployment with no funds"
  severity: M
  reasoning: "Dangerous if forgotten during migration to mainnet."
```

## 🛡️ Prevention

### Primary Defenses

- Add constructor() {_disableInitializers();} to every UUPS implementation.
- Use the initializer modifier on the initialize() function to prevent re-initialization.
- Always test deployment paths for both proxy and implementation contract.

### Additional Safeguards

- Use OpenZeppelin’s hardhat-upgrades or @openzeppelin/upgrades-core plugins which enforce initializer protection.
- Avoid deploying or exposing the implementation contract to public access unless necessary.
- Implement circuit breakers or upgrade locks in production deployments.

### Detection Methods

- Slither: missing-initializer-guard, uups-open-initialize, upgrade-takeover detectors.
- Manual review of UUPS logic for:
- Absence of initializer modifier
- Missing _disableInitializers() in constructor
- Test deployment paths with initialize() re-entry scenarios.

## 🕰️ Historical Incidents

- **Name:** Audius Governance Takeover 
- **Date:** July 2022 
- **Impact:** ~$6M stolen due to public initialize() on UUPS logic 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/audius-rekt/)


## 📚 Further Reading

- [SWC-118: Incorrect Initialization](https://swcregistry.io/docs/SWC-118) 
- [OpenZeppelin Docs – UUPS Upgradeable Contracts](https://docs.openzeppelin.com/contracts/4.x/upgradeable#uups)
- [OpenZeppelin Guide – Disable Initializers](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializers) 
- [Slither Detectors for Upgradeable Contracts](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Improperly Initialized UUPS Proxy 
severity: C
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 4.5
```

---

## 📄 Justifications & Analysis

- **Impact**: Critical — attacker can take control or destroy the proxy behavior entirely.
- **Exploitability**: Straightforward — initialize() can be called by anyone if not guarded.
- **Reachability**: High — frequent mistake in custom or forked UUPS logic.
- **Complexity**: Simple error but serious in upgradeable contexts.
- **Detectability**: Easy to detect if initializer patterns are audited.