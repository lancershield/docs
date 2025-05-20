# Incorrect Return in Assembly Leads to Malformed Output or Broken Function Logic

```YAML
id: TBA
title: Incorrect Return in Assembly Leads to Malformed Output or Broken Function Logic
severity: H
category: low-level
language: solidity
blockchain: [ethereum]
impact: Malformed data return, memory corruption, or denial of service
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-704
swc: SWC-128
```

## 📝 Description

- In Solidity, using inline assembly requires precise memory management. A common vulnerability arises when developers incorrectly use the return opcode within assembly blocks—either by miscalculating the memory offset or the return size.
- If a function ends with an incorrect return(p, n) statement in assembly, it can:
- Return garbage data
- Truncate or overflow data
- Cause abi.decode to fail or revert unexpectedly
- Return zero-length or excessive-length data leading to client deserialization errors
- This is particularly dangerous when dealing with low-level proxy fallback handlers, external call wrappers, or custom function selectors.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract BadReturn {
    function returnSomething() public pure returns (bytes memory) {
        assembly {
            let ptr := mload(0x40)
            mstore(ptr, 0x1234)
            return(ptr, 32) // ❌ Incorrect: only 2 bytes written, but returns 32
        }
    }
}
```

## 🧪 Exploit Scenario

1. A contract implements a low-level call using inline assembly.
2. It uses return(ptr, size) where size is incorrect (e.g., 32 bytes when only 4 bytes were written).
3. Downstream contracts try to decode the result using abi.decode(...), but encounter unexpected bytes.
4. The call reverts, or worse, succeeds with corrupted output, leading to broken logic, access control bypasses, or denial of service.

**Assumptions:**

- The contract uses raw return or delegatecall and manual ABI encoding.
- The developer miscalculates memory size or fails to align return data properly.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeReturn {
    function returnSomething() public pure returns (bytes memory result) {
        assembly {
            let ptr := mload(0x40)
            mstore(ptr, 0x20)              // bytes length = 32
            mstore(add(ptr, 0x20), 0x1234) // value
            mstore(0x40, add(ptr, 0x40))   // update free memory pointer
            return(ptr, 0x40)              // ✅ correct offset and size
        }
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid using low-level return in assembly unless absolutely required.
- Always write both length and data when returning dynamic types.
- Double-check return offsets and lengths against ABI expectations.

### Additional Safeguards

- Use abi.encode and abi.decode outside of assembly for safety.
- Test edge cases including malformed memory layout and zero-length returns.
- Validate return data length before decoding in the caller.

### Detection Methods

- Scan for inline assembly with return(...) usage.
- Look for mismatches between actual memory writes and declared return size.
- Tools: Slither (inline-assembly), MythX, Surya, manual review

## 🕰️ Historical Exploits

- **Name:** Gnosis Safe DelegateCall Return Bug 
- **Date:** 2021 
- **Loss:** None (patched before release) 
- **Post-mortem:** [Link to post-mortem](https://github.com/safe-global/safe-contracts/issues/250) 
- **Name:** EVM Proxy Return Bug (Ethereum Foundation) 
- **Date:** 2019 
- **Loss:** Misrouted logic due to incorrect fallback memory offset 
- **Post-mortem:** [Link to post-mortem](https://ethereum-magicians.org/t/proxies-and-return-data-bugs/3274) 
  
## 📚 Further Reading

- [SWC-128: DoS with Block Gas Limit](https://swcregistry.io/docs/SWC-128/) 
- [Solidity Docs – Inline Assembly](https://docs.soliditylang.org/en/latest/assembly.html) 
- [EVM Opcode Reference – RETURN](https://www.evm.codes/#f3?fork=merge) 
- [Gnosis Safe DelegateCall Memory Pattern](https://github.com/safe-global/safe-contracts)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Incorrect Return in Assembly Leads to Malformed Output or Broken Function Logic
severity: H
score:
impact: 4         
exploitability: 3 
reachability: 3   
complexity: 4    
detectability: 3  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Incompatible return data can lead to incorrect decoding, reverts, or fund mismanagement.
- **Exploitabilit**y: Only exploitable if attacker can trigger malformed returns or relies on corrupted output.
- **Reachability**: Common in low-level proxy logic or wrapper contracts.
- **Complexity**: High risk when dealing with dynamic types or raw memory manipulation.
- **Detectability**: Can be overlooked unless memory writes and return sizes are matched exactly.
