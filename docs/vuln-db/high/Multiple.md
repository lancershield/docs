# Multiple Constructor Schemes Bypass Initialization Logic

```YAML
id: TBA
title: Multiple Constructor Schemes Bypass Initialization Logic
severity: H
category: initialization
language: solidity
blockchain: [ethereum]
impact: Uninitialized state, unauthorized ownership, or takeover
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-665
swc: SWC-118
```

## 📝 Description

- When contracts are designed to be inherited and use multiple constructors or initialize() functions, especially in proxy deployments, inconsistencies can arise if not all initialization logic is correctly chained or enforced. 
- This vulnerability occurs when:
- Multiple constructors exist across inheritance chains.
- Initialization functions are overridden, redefined, or missed entirely.
- Base contracts silently skip logic due to missing super.initialize() calls.
- This results in critical state (like owner, admin, or token values) being unset, misconfigured, or controllable by malicious actors.

## 🚨 Vulnerable Code

```solidity

// Base contract with init logic
contract Ownable {
    address public owner;

    function initialize() public virtual {
        owner = msg.sender;
    }
}

// Derived contract forgets to call super
contract Token is Ownable {
    string public name;

    function initialize(string memory _name) public {
        name = _name;
        // ⚠️ Missing Ownable.initialize()
    }
}
```

## 🧪 Exploit Scenario

1. Proxy contract is deployed pointing to Token.
2. Admin calls Token.initialize("MyToken") to set name.
3. Ownable.initialize() is never invoked → owner remains unset.
4. Attacker later discovers this and calls Ownable.initialize() → becomes owner.
5. Attacker now has full control over token minting, transfers, or configuration.

**Assumptions:**

- Initialization logic is split across multiple contracts.
- Developer forgets to call all required initializers.
- No guard (initialized modifier) protects base contracts.

## ✅ Fixed Code

```solidity

contract Ownable {
    address public owner;
    bool internal initialized;

    function initializeOwnable() internal {
        require(!initialized, "Already initialized");
        initialized = true;
        owner = msg.sender;
    }
}

contract Token is Ownable {
    string public name;

    function initialize(string memory _name) public {
        initializeOwnable(); // ✅ Call base initializer
        name = _name;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Never rely on constructor logic in upgradable contracts.
- Always implement internal initializeXYZ() functions for base contracts.
- Use modifiers like initializer or reinitializer from OpenZeppelin.

### Additional Safeguards

- Use the Initializable pattern to track init status.
- Write unit tests for every base contract’s initialized state.
- Prevent external reinitialization of base logic.

### Detection Methods

- Check for base contracts with separate initialization logic.
- Review override chains for initialize() functions that skip super.initialize().
- Tools: Slither (missing-initializer), OpenZeppelin Upgrades Plugin, manual review

## 🕰️ Historical Exploits
 
- **Name:** Akropolis Pool Initialization Bug 
- **Date:** 2020 
- **Loss:** ~$2M 
- **Post-mortem:** [Akropolis Hack Root Cause](https://rekt.news/akropolis-rekt/) 

## 📚 Further Reading

- [SWC-118: Uninitialized State Variables](https://swcregistry.io/docs/SWC-118/) 
- [OpenZeppelin Initializable Guide](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializers) 
- [OpenZeppelin Upgradeable Contracts Best Practices](https://docs.openzeppelin.com/contracts/4.x/upgradeable) 

---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Multiple Constructor Schemes Bypass Initialization Logic
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 3   
complexity: 3     
detectability: 3  
finalScore: 3.75
```

---

## 📄 Justifications & Analysis

- **Impact**: Bypassed initialization can lead to takeover of contract ownership, minting authority, or config changes.
- **Exploitability**: Easy if an attacker identifies the missing call and initialization is public.
- **Reachability**: Proxy-based systems make this reachable post-deployment.
- **Complexity**: Moderate—developers may forget one super.initialize() call.
- **Detectability**: Requires understanding full inheritance chain, often missed in fast-paced development.


