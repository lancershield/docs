# Arbitrary Storage Write

```YAML
id: TBA
title: Arbitrary Storage Write 
baseSeverity: C
category: storage-corruption
language: solidity
blockchain: [ethereum]
impact: Overwrite critical contract variables (e.g., owner, balances, logic pointers)
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-123
swc: SWC-124
```

## 📝 Description

- Arbitrary Storage Write occurs when a smart contract allows a user to write to any storage slot, either due to:
- Misused low-level operations like `assembly` or `delegatecall`,
- Lack of bounds checking in `sstore`-like patterns
- Improper deserialization of calldata or storage pointers.
- This enables attackers to overwrite sensitive variables such as `owner`, `balances`, or `implementation` addresses in proxies, often leading to total contract compromise.

## 🚨 Vulnerable Code

```solidity
contract UnsafeStorage {
    function writeToSlot(uint256 slot, bytes32 value) external {
        assembly {
            sstore(slot, value) // ❌ unguarded arbitrary write
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker calls writeToSlot() with slot = 0 and their address as value.
2. This overwrites slot 0, which typically stores owner or implementation.
3. Attacker now has control over the contract and can drain funds, upgrade logic, or selfdestruct.

**Assumptions:**

- Storage slot layout is predictable or known.
- Function allows external control over storage index without access control or bounds checks.

## ✅ Fixed Code

```solidity

contract SafeStorage {
    address public owner;

    modifier onlyOwner() {
        require(msg.sender == owner, "Not authorized");
        _;
    }

    function updateOwner(bytes32 newOwner) external onlyOwner {
        assembly {
            sstore(0, newOwner) // ✅ Only called by authorized user
        }
    }
}
```
## 🧭 Contextual Severity

```Yaml
- context: "Default"
  severity: C
  reasoning: "Attacker can rewrite admin, logic, or balances—complete takeover or corruption."
- context: "Upgradeable proxy system with no access control"
  severity: C
  reasoning: "System can be hijacked or bricked instantly."
- context: "Properly scoped internal admin-only call"
  severity: L
  reasoning: "Still dangerous, but controlled internally with limits."
```

## 🛡️ Prevention

### Primary Defenses

- Never expose raw sstore/sload or slot-based writes to user input.
- Avoid assembly-based writes unless absolutely necessary and protected.
- Use standard high-level Solidity constructs (mapping, structs, Ownable).

### Additional Safeguards

- If using proxies, ensure implementation slots follow EIP-1967 or hardened storage patterns.
- Use access control (onlyOwner, onlyProxyAdmin) rigorously.
- Validate and sanitize any index or offset used to write to storage.

### Detection Methods

- Slither: low-level-write, arbitrary-sstore, dangerous-assembly detectors.
- Manual code audit for sstore, delegatecall, or storage[] usage.
- Formal analysis or fuzzing with randomized slot access and input injection.

## 🕰️ Historical Exploits

- **Name:** Multichain Storage Collision 
- **Date:** 2023 
- **Loss:** ~$1.9M 
- **Post-mortem:** [Link to post-mortem](https://slowmist.medium.com) 
  
## 📚 Further Reading

- [SWC-124: Arbitrary Storage Write](https://swcregistry.io/docs/SWC-124) 
- [Solidity Docs – Storage Layout](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html)
- [OpenZeppelin – Proxy Storage Slots](https://docs.openzeppelin.com/contracts/4.x/api/proxy#ERC1967Upgrade)
- [EIP-1967 – Standard Proxy Storage Slots](https://eips.ethereum.org/EIPS/eip-1967) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Arbitrary Storage Write 
severity: C
score:
impact: 5        
exploitability: 4
reachability: 4   
complexity: 3     
detectability: 4  
finalScore: 4.4
```

---

## 📄 Justifications & Analysis

- **Impact**: Overwrites critical storage slots (e.g., owner, implementation, balances).
- **Exploitability**: A single crafted slot and value combo can hijack the contract.
- **Reachability**: Often exposed via developer backdoors or debug features.
- **Complexity**: Medium – attacker must understand the storage slot layout.
- **Detectability**: Highly detectable in audits with proper Slither rules or grep-based search.