# Unchecked transfer

```YAML
id: LS47M
title: Unchecked transfer
baseSeverity: M
category: ether-transfer
language: solidity
blockchain: [ethereum]
impact: Funds may be lost or logic may silently fail
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.0"]
cwe: CWE-755
swc: SWC-134
```

## 📝 Description

- In Solidity, the native transfer() function is often used to send Ether. 
- While transfer() automatically reverts on failure, many developers use low-level calls (call{value: ...}("")) to bypass gas limits or support smart contract recipients.
- When using call{value: ...}("") without checking the return value, Ether transfers can silently fail, leading to:
- Undetected fund loss
- Broken payout systems
- Reentrancy windows if misused inside external calls
- If a transfer fails (e.g., recipient reverts, runs out of gas, or is a contract that disallows receiving Ether), the contract may continue execution assuming success, leading to loss of accounting or funds stuck forever.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract UncheckedTransfer {
    function payout(address payable recipient) public {
        // ❌ Ignoring return value of low-level call
        recipient.call{value: 1 ether}(""); 
    }

    receive() external payable {}
}
```

## 🧪 Exploit Scenario

1. Contract owner tries to payout rewards to recipient.
2. recipient is a contract that reverts in its fallback or uses too much gas.
3. The call fails, but since the result isn’t checked, the contract continues execution.
4. The reward is marked as paid, but no Ether is transferred → funds remain stuck in the contract.

**Assumptions:**

- Developer uses low-level transfer methods.
- No checks are performed on return value or error handling.
- Recipient behavior is not guaranteed (e.g., untrusted user or DAO).

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeTransfer {
    function payout(address payable recipient) public {
        (bool success, ) = recipient.call{value: 1 ether}("");
        require(success, "Transfer failed");
    }

    receive() external payable {}
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Can silently fail Ether transfers without warning or rollback."
- context: "Public protocol handling open recipient lists"
  severity: H
  reasoning: "Attackers can cause denial of funds or corrupt payment logic."
- context: "Internal system with known addresses"
  severity: L
  reasoning: "Risk is lower if all recipient contracts are audited and predictable."
```

## 🛡️ Prevention

### Primary Defenses

- Always check the return value of call{value: ...}("").
- Use require(success, "...") to revert on failure.

### Additional Safeguards

- Consider using send() or transfer() where applicable for simple transfers.
- Avoid Ether transfers to contracts that you do not control unless needed.
- If non-reverting logic is required, log failed attempts and allow retries.

### Detection Methods

- Check for call{value:} with ignored return values.
- Search for missing require(success) patterns after low-level calls.
- Tools: Slither (unchecked-lowlevel-call), MythX, manual audit

## 🕰️ Historical Exploits

- **Name:** King of the Ether 
- **Date:** 2016 
- **Loss:** $10K permanently locked 
- **Post-mortem:** [Link to post-mortem](https://ethereum.stackexchange.com/questions/19341/what-happened-with-the-king-of-the-ether-throne-contract) 

## 📚 Further Reading

- [SWC-104: Unchecked Call Return Value](https://swcregistry.io/docs/SWC-104/) 
- [Solidity Docs – call, delegatecall, and staticcall](https://docs.soliditylang.org/en/latest/control-structures.html#external-function-calls) 
- [Slither Detector – Unchecked Low-Level Calls](https://github.com/crytic/slither/wiki/Detector-Documentation#unchecked-low-level-calls)

--- 
  
## ✅ Vulnerability Report

```markdown
id: LS47M
title: Unchecked transfer
severity: M
score:
impact: 4         
exploitability: 3 
reachability: 4   
complexity: 2    
detectability: 4  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Funds may be marked as transferred but never reach the recipient.
- **Exploitability**: If the recipient reverts or breaks fallback logic, funds remain stuck.
- **Reachability**: Any contract using low-level Ether transfers is exposed.
- **Complexity**: Very easy to introduce; one-line mistake.
- **Detectability**: Often overlooked due to over-trust in call.