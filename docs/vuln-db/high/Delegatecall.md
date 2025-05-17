# Delegatecall to Self Causing State Corruption 

```YAML
id: TBA
title: Delegatecall to Self Causing State Corruption or Bypass of Access Controls
severity: H
category: delegatecall
language: solidity
blockchain: [ethereum]
impact: Logic reuse bypasses internal access checks or corrupts storage layout
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-841
swc: SWC-112
```

## 📝 Description

- Delegatecall to self occurs when a contract makes a `delegatecall` to its own address (i.e., `address(this)`) or an implementation contract does so within a proxy context. 
- This breaks assumptions about access controls, context variables like `msg.sender`, and storage layout, because:
`delegatecall` preserves the caller’s context and storage
- But functions executed via `delegatecall(address(this), ...)` run against the proxy’s storage, not the logic contract’s.
- Resulting in storage corruption, logic bypass, or privilege escalation.
- This is particularly dangerous in upgradable contracts where the logic contract is reused across multiple proxies.

## 🚨 Vulnerable Code

```solidity
contract Vulnerable {
    address public owner;

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function setOwner(address newOwner) public onlyOwner {
        owner = newOwner;
    }

    function proxyCall(bytes memory data) public {
        // ❌ Dangerous self-delegatecall bypasses modifiers
        (bool success, ) = address(this).delegatecall(data);
        require(success, "delegatecall failed");
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker crafts a call to setOwner(attacker) and passes it to proxyCall().
2. delegatecall(address(this), data) executes setOwner() in the proxy’s context, not checking msg.sender == owner (modifier runs in context of caller, not target).
3. owner is overwritten with attacker address.
4. Attacker gains control of the contract.

**Assumptions**:

- The contract uses delegatecall(address(this), ...) for internal calls.
- Critical functions like setOwner() have modifier checks assuming normal calls.

## ✅ Fixed Code

```solidity
contract Safe {
    address public owner;

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function setOwner(address newOwner) public onlyOwner {
        owner = newOwner;
    }

    function proxyCall(bytes memory data) public onlyOwner {
        (bool success, ) = address(this).call(data); // ✅ Use .call if logic is safe and controlled
        require(success, "call failed");
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid using delegatecall to address(this) unless strictly necessary and fully understood.
- Use .call() or direct internal function calls for logic reuse.
- If delegatecall is used, isolate privileged functions behind stricter access controls.

### Additional Safeguards

- Mark all privileged functions as external and disallow delegatecall() access.
- Detect and reject delegatecall-to-self attempts by checking address(this) == implementation.

### Detection Methods

- Slither: dangerous-low-level, self-delegatecall, and context-confusion detectors.
- Manual inspection of any delegatecall using address(this).
- Use invariant testing: assert msg.sender == tx.origin || isWhitelisted(msg.sender) in sensitive paths.

## 🕰️ Historical Exploits

- **Name:** Parity Wallet Delegatecall Bug 
- **Date:** 2017 
- **Loss:** ~$150M frozen 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert-2/) 

## 📚 Further Reading

- [SWC-112: Delegatecall to Untrusted Callee](https://swcregistry.io/docs/SWC-112) 
- [Solidity Docs – delegatecall Caveats](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#delegatecall-callcode-and-libraries) 
- [OpenZeppelin – Upgradeable Contract Patterns](https://docs.openzeppelin.com/upgrades-plugins/1.x/) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Delegatecall to Self Causing State Corruption 
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

- **Impact**: Allows unauthorized control of contract by rewriting storage or bypassing access checks.
- **Exploitability**: Requires minimal code interaction if a delegatecall(this, ...) exists.
- **Reachability**: Found in upgradable patterns, fallback routers, or generic relays.
- **Complexity**: Moderate — attacker needs to craft payload and know modifiers won't trigger as expected.
- **Detectability**: Easily caught via static analysis or known delegatecall patterns.