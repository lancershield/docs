# Cross-Contract Execution Risks

```YAML
id: TBA
title: Cross-Contract Execution Risks from Untrusted External Calls
severity: H
category: external-call
language: solidity
blockchain: [ethereum]
impact: Unexpected state changes, reentrancy, or asset loss
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-710
swc: SWC-104
```


## 📝 Description

- Cross-contract execution risks arise when a contract makes external calls to **untrusted contracts**, assuming they will behave safely. These risks include:
- **Reentrancy** attacks if state updates happen after the call.
- **DoS** via failed calls or unhandled reverts.
- **Unexpected behavior** if the callee implements malicious logic.
- Smart contracts must treat any external call—including to tokens, oracles, or utility contracts—as potentially hostile.

## 🚨 Vulnerable Code

```solidity
contract Auction {
    address public highestBidder;
    uint256 public highestBid;

    function bid() external payable {
        require(msg.value > highestBid, "Low bid");

        // Return ETH to previous bidder
        if (highestBidder != address(0)) {
            payable(highestBidder).call{value: highestBid}(""); // ❌ unsafe external call
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker becomes the highest bidder.
2. Contract tries to refund the attacker’s ETH on the next bid.
3. Attacker’s fallback function re-enters the bid() logic before state is updated.
4. Multiple states are corrupted, or the attacker prevents further bids (DoS or drain).

**Assumptions:**

- Caller is a contract with a fallback or malicious logic.
- No protection like ReentrancyGuard or proper state-before-call pattern.

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SafeAuction is ReentrancyGuard {
    address public highestBidder;
    uint256 public highestBid;
    mapping(address => uint256) public pendingReturns;

    function bid() external payable nonReentrant {
        require(msg.value > highestBid, "Low bid");

        if (highestBidder != address(0)) {
            pendingReturns[highestBidder] += highestBid;
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    function withdraw() external {
        uint256 amount = pendingReturns[msg.sender];
        require(amount > 0, "Nothing to withdraw");
        pendingReturns[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use Checks-Effects-Interactions pattern (update state before external calls).
- Prefer pull over push payment models (e.g., withdraw() instead of send()).
- Use ReentrancyGuard to block re-entrant logic paths.

### Additional Safeguards

- Verify that the callee is a trusted contract (whitelist).
- Avoid low-level calls (call()) unless strictly necessary.
- Wrap external calls with try/catch if supported (Solidity ≥ 0.6.0).

### Detection Methods

- Slither: reentrancy-eth, low-level-calls, external-calls detectors.
- Manual audit of external interactions, especially to user-controlled contracts.
- Dynamic testing using reentrancy simulation contracts.

## 🕰️ Historical Exploits

- **Name:** The DAO Hack 
- **Date:** 2016-06-17 
- **Loss:** ~$60M 
- **Post-mortem:** [Link to post-mortem](https://blog.slock.it/the-dao-hack-explained-62429dbabf62) 
  

## 📚 Further Reading

- [SWC-104: Unchecked Call Return Value](https://swcregistry.io/docs/SWC-104) 
- [OpenZeppelin – ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) 
- [Trail of Bits – Cross-Contract Communication](https://github.com/trailofbits/publications/blob/master/reviews/Compound-2018-10.pdf)
  
--- 
## ✅ Vulnerability Report 


```markdown
id: TBA
title: Cross-Contract Execution Risks 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 3  
finalScore: 4.15
```


---

## 📄 Justifications & Analysis

- **Impact**: A malicious callee can reenter, block, or manipulate the caller contract.
- **Exploitability**: Any attacker with a contract can exploit it if protections are missing.
- **Reachability**: Common in systems handling refunds, payouts, and oracle responses.
- **Complexity**: Low-to-moderate effort depending on fallback logic used.
- **Detectability**: Often flagged in security scans, but still common in audits.
