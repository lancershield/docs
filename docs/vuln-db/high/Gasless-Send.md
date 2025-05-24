# Gasless Send

```YAML
id: TBA
title: Gasless Send
severity: H
category: ether-transfer
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: ETH transfers may silently fail or revert due to limited gas forwarding
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-754
swc: SWC-113
```

## 📝 Description

- A Gasless Send vulnerability occurs when a contract uses Solidity's address.send() or address.transfer() to send ETH, which forwards only 2300 gas. 
- If the recipient is a contract with a fallback or receive function requiring more gas, the transfer silently fails, breaking core flows such as refunds, withdrawals, or payments.
- send() returns a boolean and does not revert, so failures are easy to ignore.
- transfer() does revert, but both restrict gas and are not suitable for cross-contract ETH transfers.
- Contracts like multisigs, upgradable proxies, or fallback-heavy wallets require more than 2300 gas, and fail under these methods.
- This vulnerability leads to:
- Denial of ETH to contract-based users
- Trapped funds or inconsistent state
- Refund failures in auctions, staking, or presales

## 🚨 Vulnerable Code

```solidity

function refund(address recipient) external {
    require(address(this).balance >= 1 ether, "Insufficient");
    recipient.send(1 ether); // ❌ silently fails if recipient is a contract needing >2300 gas
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A user participates in a presale and later requests a refund.
2. The contract tries to use send() to refund ETH.
3. The user’s address is a contract (e.g., Gnosis Safe) that has a fallback consuming more than 2300 gas.
4. The send() fails silently and returns false.
5. The contract logic continues without reverting or informing the user.
6. Funds remain stuck and unrecoverable from the user's perspective.

**Assumptions:**

- Contract uses send() or transfer() instead of call.
- Recipient may be a smart contract wallet or proxy requiring gas.
- No event is emitted or error handling performed on failure.

## ✅ Fixed Code

```solidity

function refund(address payable recipient) external {
    (bool success, ) = recipient.call{value: 1 ether}("");
    require(success, "ETH refund failed");
}
```

## 🛡️ Prevention

### Primary Defenses

- Never use send() or transfer() in modern Solidity (post-0.6.0).
- Use call{value: amount}("") for ETH transfers with proper error handling.
- Emit events for every ETH transfer attempt and failure.

### Additional Safeguards

- Validate recipients (e.g., known EOAs vs. contracts).
- Add fallback withdrawal or reclaim mechanism.
- Design workflows to support both pull and push payment models.

### Detection Methods

- Static analysis for send() or transfer() usage.
- Fuzz testing with contracts requiring >2300 gas.
- Tools: Slither (dangerous-low-level), Foundry reentrancy/invariant tests

## 🕰️ Historical Exploits

- **Name:** Ethereum Classic DAO Refund Failures 
- **Date:** 2016 
- **Loss:** ~$6M 
- **Post-mortem:** [Link to post-mortem](https://ethereum.org/en/history/#the-dao) 
 
## 📚 Further Reading

- [SWC-113: DoS with Failed Call](https://swcregistry.io/docs/SWC-113)  
- [Solidity Docs – Sending Ether](https://docs.soliditylang.org/en/latest/security-considerations.html#sending-and-receiving-ether) 
- [OpenZeppelin Security Guidelines](https://docs.openzeppelin.com/contracts/4.x/api/utils#Address-sendValue-address-uint256)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Gasless Send
severity: H
score:
impact: 4  
exploitability: 4  
reachability: 5  
complexity: 1  
detectability: 5  
finalScore: 3.95
```

---

## 📄 Justifications & Analysis

- **Impact**: Blocks ETH transfers to valid users; may result in user fund loss or refund denial.
- **Exploitability**: No active exploit needed—just use a contract wallet with >2300 gas needs.
- **Reachability**: Common in payout, withdrawal, and presale refund logic.
- **Complexity**: Extremely simple; accidental user behavior can trigger it.
- **Detectability**: Highly detectable via Slither, Linter, and Solidity documentation best practices.
