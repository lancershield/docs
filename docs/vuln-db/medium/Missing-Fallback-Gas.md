# Missing Fallback Gas

```YAML
id: LS20M
title: Missing Fallback Gas 
baseSeverity: M
category: ether-transfer
language: solidity
blockchain: [ethereum]
impact: Failed payments, broken integrations, or stuck funds
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-754
swc: SWC-134
```

## 📝 Description

- In Solidity, sending Ether using address.transfer() or address.send() forwards only 2300 gas, which is not enough to run complex fallback logic in recipient contracts. 
- This is a deliberate gas limit introduced to prevent reentrancy, but when used in modern contracts, it may lead to:
- Unexpected failed transfers when the recipient is a contract with logging, emitting events, or basic state changes in its receive() or fallback() functions.
- Stuck funds, especially in reward, split, or withdrawal patterns.
- Untraceable silent failures if the return value from send() is ignored.
- Modern Solidity best practices suggest using call{value: amount}("") instead, which allows full gas forwarding and explicit failure handling.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Rewarder {
    function reward(address recipient, uint256 amount) external {
        payable(recipient).transfer(amount); // ❌ only 2300 gas forwarded
    }

    receive() external payable {
        // Contract logic: logs or state update
    }
}
```

## 🧪 Exploit Scenario

1. Alice deploys a smart contract that includes a receive() function with a small event emit.
2. Bob interacts with a dApp that uses transfer() to send rewards to Alice's contract.
3. The transfer fails due to gas stipend limitation.
4. Bob’s transaction reverts or the app behaves inconsistently (e.g., funds deducted but not received).

**Assumptions:**

- Recipient contract has logic in receive() or fallback().
- Ether is sent using transfer() or send().

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeRewarder {
    function reward(address recipient, uint256 amount) external {
        (bool sent, ) = payable(recipient).call{value: amount}(""); // ✅ full gas forwarded
        require(sent, "Transfer failed");
    }

    receive() external payable {
        // Contract logic runs safely
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Transfer failures cause inconsistencies but no direct theft."
- context: "High-throughput airdrop or treasury contract"
  severity: H
  reasoning: "Numerous silent failures may accumulate, causing major fund misallocation."
- context: "Minimal contract-to-contract interaction"
  severity: L
  reasoning: "Low risk if interacting only with EOAs or gasless contracts."
```

## 🛡️ Prevention

### Primary Defenses

- Replace all uses of transfer() and send() with call{value: ...}("").
- Add require(sent, "Transfer failed") to enforce success handling.

### Additional Safeguards

- In protocols distributing Ether, prefer pull payments (e.g., withdraw()) over push payments.
- If limiting reentrancy is a concern, combine call{value:...} with nonReentrant protection.

### Detection Methods

- Search for transfer() and send() usage.
- Static analysis for gas-forwarding methods.
- Tools: Slither (dangerous-usage-of-transfer), MythX, Solhint

## 🕰️ Historical Exploits

- **Name:** King of the Ether Throne 
- **Date:** 2016 
- **Loss:** Approximately 1,000 ETH  
- **Post-mortem:** [Link to post-mortem](https://ethereum-contract-security-techniques-and-tips.readthedocs.io/en/latest/known_attacks/) 
- **Name:** Uniswap v1 Token Transfer Failures 
- **Date:** 2018 
- **Loss:** Estimated $50,000  
- **Post-mortem:** [Link to post-mortem](https://stackoverflow.com/questions/74930609/solidity-fallback-function-gas-limit)
  
## 📚 Further Reading

- [SWC-134: Unhandled Exceptions](https://swcregistry.io/docs/SWC-134/) 
- [Solidity Docs – Transfer vs Send vs Call](https://docs.soliditylang.org/en/latest/security-considerations.html#use-call-value-where-appropriate) 
- [OpenZeppelin – sendValue Utility](https://docs.openzeppelin.com/contracts/4.x/api/utils#Address-sendValue-address-payable-uint256-) 
  
## ✅ Vulnerability Report 

```markdown
id: LS20M
title: Missing Fallback Gas 
severity: M
score:
impact: 3        
exploitability: 2 
reachability: 4 
complexity: 2  
detectability: 5  
finalScore: 3.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Users may not receive funds due to silent failures in transfer logic.
- **Exploitability**: Not directly exploitable, but can cause denial-of-service or fairness issues.
- **Reachability**: Common wherever Ether is sent to arbitrary user addresses.
- **Complexity**: Trivial error; many old tutorials still use transfer().
- **Detectability**: Easily flagged with automated tools or basic linting.