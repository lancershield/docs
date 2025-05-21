# Unchecked send

```YAML
id: TBA
title: Unchecked send
severity: M
category: unchecked-return
language: solidity
blockchain: [ethereum]
impact: Funds stuck, inconsistent state updates, or silent denial-of-service
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-252
swc: SWC-104
```

## 📝 Description

- The low-level Solidity function address.send() transfers exactly 2300 gas along with Ether and returns a boolean indicating success or failure. 
- If the return value is not checked, the contract may:
- Assume the transfer succeeded when it silently failed
- Update storage state (e.g., marking a payout) even though the user received no ETH
- Become vulnerable to logic desynchronization, especially in payment flows, auctions, or refund mechanisms
- This leads to fragile user experience, fund loss, or denial of service, particularly when interacting with contracts that have non-trivial fallback logic or receive hooks.

## 🚨 Vulnerable Code

```solidity
pragma solidity ^0.8.0;

contract Auction {
    address public highestBidder;
    uint256 public highestBid;

    function refund(address payable loser, uint256 amount) internal {
        // ❌ Ignores return value of send
        loser.send(amount);
    }
}
```

## 🧪 Exploit Scenario

1. A user is outbid in an auction and is due for a refund.
2. The contract calls loser.send(amount) without checking the result.
3. The recipient is a contract with a complex fallback function, so send() fails.
4. The user's refund is silently lost, and the contract believes the refund succeeded.
5. The user is denied access to funds and cannot re-enter the auction or claim loss.

**Assumptions:**

- Logic continues after send() even if it fails.
- No state rollback or failure logging is implemented.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeAuction {
    function safeRefund(address payable loser, uint256 amount) internal {
        bool sent = loser.send(amount);
        require(sent, "Refund failed"); // ✅ Revert on failure
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Never ignore the return value of send() or call.
- Use require(success) to enforce transfer success or fallback logic.

### Additional Safeguards

- Consider pull-payment patterns (withdraw() by user) to avoid sending ETH directly.
- Use ReentrancyGuard to protect call{value:} logic in public functions.

### Detection Methods

- Look for send() calls without require() or condition handling.
- Search for functions that update state after unchecked send() calls.
- Tools: Slither (unchecked-send), MythX, Surya, manual audit

## 🕰️ Historical Exploits

- **Name:** King of the Ether Throne 
- **Date:** 2016 
- **Loss:** ~10,000 ETH permanently stuck due to fallback execution and send limitations 
- **Post-mortem:** [Link to post-mortem](https://ethereum.stackexchange.com/questions/19341/what-happened-with-the-king-of-the-ether-throne-contract)

📚 Further Reading

- [SWC-104: Unchecked Call Return Value](https://swcregistry.io/docs/SWC-104/) 
- [Solidity Docs – send() vs. call()](https://docs.soliditylang.org/en/latest/security-considerations.html#sending-and-receiving-ether)
- [Slither Detector – Unchecked Low-Level Calls](https://github.com/crytic/slither/wiki/Detector-Documentation#unchecked-low-level-calls) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Unchecked send
severity: M
score:
impact: 3         
exploitability: 2 
reachability: 4   
complexity: 1     
detectability: 5  
finalScore: 2.9
```

---

## 📄 Justifications & Analysis

- **Impact**: May result in funds lost or users blocked from receiving payouts.
- **Exploitability**: Depends on the recipient contract, but trivial to simulate.
- **Reachability**: Frequent in auctions, withdrawals, and reward systems.
- **Complexity**: Caused by ignoring send() behavior and gas limits.
- **Detectability**: Easily identified by linters and static analyzers.