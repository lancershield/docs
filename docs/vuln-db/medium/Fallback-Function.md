# Fallback Function Misuse

```YAML

id: TBA
title: Fallback Function Misuse Leading to Asset Loss
severity: M
category: fallback-function
language: solidity
blockchain: [ethereum]
impact: Unintended execution or asset loss via improperly handled fallback
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-284
swc: SWC-115
```

## 📝 Description

- Fallback function misuse occurs when the contract's `fallback()` or `receive()` functions unintentionally accept Ether or execute logic due to missing or incorrect function selectors, misrouted calls, or default catch-all behavior.
- This can lead to:
- Accepting funds when not intended.
- Unexpected logic execution via `delegatecall`.
- Confusing or inconsistent contract behavior in upgradeable systems.

## 🚨 Vulnerable Code

```solidity
contract MisusedFallback {
    address public owner;

    fallback() external payable {
        owner = msg.sender; // ❌ dangerous logic in fallback
    }
}

```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker calls the contract with a random function selector (nonexistent).
2. The fallback() function is triggered and sets themselves as owner.
3. Attacker now controls the contract without calling any known function.
4. If fallback had balance updates or critical logic, it could be misused repeatedly.

**Assumptions:**

- Fallback or receive() function includes critical logic.
- No function selector guards or input sanitization.

## ✅ Fixed Code

```solidity

// Safe fallback — only logs or rejects calls
fallback() external payable {
    revert("Function does not exist");
}

// If ETH should be accepted
receive() external payable {
    // Accept ETH only
}
```

## 🛡️ Prevention

### Primary Defenses

- Keep fallback functions minimal — preferably just revert() or emit logs.
- Do not include critical logic in fallback or receive() functions.
- Clearly separate logic that handles Ether (use receive()) from unknown call logic (fallback).

### Additional Safeguards

- Use interfaces for external calls to avoid triggering fallback.
- Implement explicit function selectors on proxies and dispatchers.
- Always test with unknown calls and raw call() invocations.

### Detection Methods

- Slither: dangerous-fallback, low-level-calls detectors.
- Manual code review of fallback() and receive() logic.
- Static/dynamic analysis for function selector collisions.

## 🕰️ Historical Exploits

- **Name:** Rubixi (Dynamic Pyramid) Exploit
- **Date:** 2016-05
- **Loss:** N/A (Unintended ownership assignment via fallback)
- **Post-mortem:** [Link to post-mortem](https://medium.com/@PracticalDev/the-rubixi-bug-a-smart-contract-vulnerability-with-an-interesting-history-c06c41f5a6b8)

- **Name:** Upbit Token Proxy Admin Exposure
- **Date:** 2022-01-18
- **Loss:** N/A (Whitehat discovery)
- **Post-mortem:** [Link to post-mortem](https://twitter.com/pcaversaccio/status/1483800152813000705)

## 📚 Further Reading

- [SWC-115: Misuse of Fallback Function](https://swcregistry.io/docs/SWC-115)
- [Solidity Docs – Fallback and Receive Functions](https://docs.soliditylang.org/en/latest/contracts.html#fallback-function)
- [OpenZeppelin Security Guidelines](https://docs.openzeppelin.com/contracts/4.x/api/security)

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Fallback Function Misuse
severity: M
score:
impact: 3  
exploitability: 3
reachability: 4  
complexity: 2  
detectability: 4  
finalScore: 3.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Attacker may gain access or trigger undesired state changes silently.
- **Exploitability**: Can be invoked by sending transactions to invalid function selectors.
- **Reachability**: Triggered automatically on incorrect function calls or ETH sends.
- **Complexity**: Simple to exploit—only requires a basic transaction or fallback trigger.
- **Detectability**: Commonly flagged by Slither and caught in audits if fallback is nontrivial.
