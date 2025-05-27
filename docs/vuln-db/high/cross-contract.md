# Cross-Contract Execution 

```YAML
id: TBA
title: Cross-Contract Execution 
baseSeverity: H
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

## 🧭 Contextual Severity

```yaml
- context: "Unvalidated call to arbitrary target with full calldata"
  severity: H
  reasoning: "Allows attacker to control execution, escalate privileges, or drain funds"
- context: "Whitelisted call targets but unchecked return values"
  severity: M
  reasoning: "Logic may still be corrupted due to silent call failure"
- context: "Call target is immutable and audited"
  severity: L
  reasoning: "Low risk if trust boundaries are respected"
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

- **Name:** Nomad Bridge Exploit 
- **Date:** 2022-08-01 
- **Loss:** Approximately $190 million 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/nomad-rekt/) 
- **Name:** Cross-Chain Bridge Exploits 
- **Date:** 2023 
- **Loss:** Over $2 billion across multiple incidents 
- **Post-mortem:** [Link to post-mortem](https://coinlaw.io/smart-contract-security-risks-and-audits-statistics/) 

## 📚 Further Reading

- [CrossFuzz: Cross-Contract Fuzzing for Smart Contract Vulnerability Detection](https://www.sciencedirect.com/science/article/pii/S0167642323001582) 
- [Smart Contracts Security Challenges Explained](https://www.lcx.com/smart-contracts-security-challenges-explained/) 
 
--- 

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Cross-Contract Execution
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
