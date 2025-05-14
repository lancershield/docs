# Unused Modifiers

```YAML

id: TBA
title: Unused Modifiers Leading to Assumed but Missing Access Control
severity: M
category: access-control
language: solidity
blockchain: [ethereum]
impact: Critical functions may be callable by anyone despite intended restrictions
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-862
swc: SWC-100
```

## 📝 Description

- Unused modifiers are declared in smart contracts (e.g., `onlyOwner`, `nonReentrant`, `whenNotPaused`) but never actually applied to any functions. 
- This creates a false sense of security, especially if developers assume the modifier is in effect. 
- If critical functions like `mint()`, `upgrade()`, or `withdraw()` lack necessary modifiers, any user may call them, leading to privilege escalation, asset theft, or protocol compromise.

## 🚨 Vulnerable Code

```solidity
contract BrokenAccess {
    address public owner;

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    // ❌ No modifier applied — function is publicly callable
    function mint(address to, uint256 amount) public {
        // mint tokens to 'to'
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Developer defines onlyOwner and assumes mint() uses it.
2. Auditor or contributor overlooks the missing modifier.
3. Attacker calls mint() directly and mints unlimited tokens to themselves.
4. Token supply is corrupted, price crashes, protocol is exploited.

**Assumptions:**

- Modifiers exist but are unused.
- Critical functions are not protected by any other mechanism (e.g., role-based access control).

## ✅ Fixed Code

```solidity

function mint(address to, uint256 amount) public onlyOwner {
    // ✅ Now properly restricted to owner
}
```

## 🛡️ Prevention

### Primary Defenses

- Always verify that every function needing access control explicitly applies the required modifier.
- Adopt tooling or linting that flags declared-but-unused modifiers.
- Include automated role enforcement via OpenZeppelin's AccessControl or Ownable.

### Additional Safeguards

- Use code reviews to confirm modifier coverage on privileged logic.
- Include unit tests that ensure unauthorized users are rejected from sensitive calls.
- Consider restricting critical functions to external or onlyRole() as an extra layer.

### Detection Methods

- Slither: unused-modifier, missing-access-control, inconsistent-access-pattern detectors.
- Manual review comparing modifier declarations with function usages.
- Static analyzers like MythX and Oyente can catch unsafe access patterns.

## 🕰️ Historical Exploits

- **Name:** YAM Finance Unchecked Access 
- **Date:** 2020 
- **Loss:** Protocol halted due to privilege misconfiguration 
- **Post-mortem:** [Link](https://medium.com/yam-finance/yam-finance-post-mortem-53f8b0b3f1af) 


## 📚 Further Reading

- [SWC-100: Function Default Visibility](https://swcregistry.io/docs/SWC-100) 
- [OpenZeppelin AccessControl Patterns](https://docs.openzeppelin.com/contracts/4.x/access-control) 
- [Slither: Detecting Unused Modifiers](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Unused Modifiers Leading to Assumed but Missing Access Control
severity: M
score:
impact: 4         
exploitability: 3 
reachability: 3   
complexity: 1     
detectability: 5  
finalScore: 3.3
```


---

## 📄 Justifications & Analysis

- **Impact**: Unprotected privileged functions can result in token inflation, asset loss, or unauthorized upgrades.
- **Exploitability**: Moderate; attacker just calls an unprotected function.
- **Reachability**: High; happens in any logic using custom access control patterns.
- **Complexity**: Low — caused by developer oversight.
- **Detectability**: Very high — static analyzers like Slither reliably detect it.