# Return Bomb

```YAML
id: TBA
title: Return Bomb 
severity: H
category: denial-of-service
language: solidity
blockchain: [ethereum]
impact: DoS via oversized return data and gas depletion
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-400
swc: SWC-113
```

## 📝 Description

- A Return Bomb is a denial-of-service (DoS) vulnerability where an attacker contract returns an excessively large amount of data (e.g., in fallback() or receive()), causing the caller contract to exhaust gas when trying to decode or copy the data using high-level Solidity functions such as:
- contract.functionCall(...)
- interface.method()
- abi.decode(...)
- Because Solidity will automatically copy return data into memory for decoding, attacker contracts can force OOG (out-of-gas) errors, making the caller revert every time it interacts with the malicious target.

## 🚨 Vulnerable Code

```solidity
pragma solidity ^0.8.0;

interface ITarget {
    function getData() external view returns (bytes memory);
}

contract ReturnBombCaller {
    function callTarget(address target) external view returns (bytes memory) {
        return ITarget(target).getData();  // ❌ vulnerable to return bomb
    }
}
```

## 🧪 Exploit Scenario

1. An attacker deploys a contract where getData() returns 10MB of data.
2. A dApp uses a try/catch or external interface call to getData() expecting a small response.
3. The attacker calls callTarget(attacker_contract) via a frontend or aggregator.
4. The entire transaction runs out of gas due to memory allocation during return data copying.
5. The protocol is effectively DoS'd if it automatically processes many such targets or relies on the external return value.

**Assumptions:**

- Call is made via high-level interface expecting a bounded response.
- No gas limits or return-size checks are enforced on the low-level call.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract ReturnBombSafeCaller {
    function safeCall(address target) external view returns (bytes memory) {
        (bool success, bytes memory data) = target.staticcall{gas: 50000}(abi.encodeWithSignature("getData()"));
        require(success, "Call failed");
        require(data.length <= 1024, "Return bomb detected"); // ✅ Limit data size
        return data;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Apply gas caps when using .call or .staticcall to untrusted contracts.
- Check returnData.length before using abi.decode() or returning directly.

### Additional Safeguards

- Use try/catch for external calls but also apply length checks inside the try.
- If working with known interfaces, whitelist bytecode size or audited addresses.

### Detection Methods

- Review all external interface calls that return dynamic arrays or bytes.
- Audit external calls without gas or data length limits.
- Tools: Slither (unbounded-return), custom static analysis, fuzzing for oversized return

## 🕰️ Historical Exploits

- **Name:** KingOfTheHill Return Bomb Vulnerability 
- **Date:** 2021 
- **Loss:** Potential denial-of-service due to excessive return data causing out-of-gas errors 
- **Post-mortem:** [Link to post-mortem](https://github.com/ethereum/solidity/issues/12306) 
  
## 📚 Further Reading

- [SWC-113: DoS with Block Gas Limit](https://swcregistry.io/docs/SWC-113/) 
- [Solidity Docs – External Function Calls](https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions)  
- [Slither – Unbounded Return Data](https://github.com/crytic/slither/wiki/Detector-Documentation#unbounded-return)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Return Bomb 
severity: H
score:
impact: 4 
exploitability: 3 
reachability: 3 
complexity: 2  
detectability: 3  
finalScore: 3.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Entire protocols can be halted or reverted due to call-based return bombs.
- **Exploitability**: Any attacker can deploy a contract that returns 10MB data cheaply.
- **Reachability**: Frequent in DeFi aggregator wrappers, multi-call tools, and relays.
- **Complexity**: Requires knowledge of memory allocation behavior and target contract patterns.
- **Detectability**: Subtle—often missed in surface-level audits if calls seem harmless.