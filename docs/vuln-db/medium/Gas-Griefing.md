# Gas Griefing 

```YAML
id: TBA
title: Gas Griefing via Forced Reverts or Refund Denial
severity: M
category: gas-griefing
language: solidity
blockchain: [ethereum]
impact: Denial of service, refund blockage, or UX degradation
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<0.8.21"]
cwe: CWE-400
swc: SWC-136
```
## 📝 Description

- Gas Griefing occurs when an attacker exploits gas limitations or fallback behavior to make a function unusable or undesirably expensive. 
- This can happen when:
- A contract calls untrusted external addresses that revert or consume all gas.
- Gas stipends (e.g., `2300` gas in `transfer`) are insufficient for smart contract recipients.
- Refunds, loops, or bulk processing are blocked due to one recipient failing or consuming all gas.
- This leads to denial of service (DoS) or degraded user experience, especially in payout or voting systems.

## 🚨 Vulnerable Code

```solidity
contract Airdrop {
    mapping(address => bool) public claimed;

    function claim(address[] calldata recipients) external {
        for (uint i = 0; i < recipients.length; i++) {
            if (!claimed[recipients[i]]) {
                payable(recipients[i]).transfer(1 ether); // ❌ fails if any recipient reverts
                claimed[recipients[i]] = true;
            }
        }
    }

    receive() external payable {}
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker deploys a contract with a fallback function that always reverts or consumes all gas.
2. Attacker gets included in the recipients list for a bulk payout.
3. When claim() is called, the call to the attacker's address fails or runs out of gas.
4. The entire loop or transaction fails, preventing anyone from receiving their payout.

**Assumptions:**

- The loop or execution does not handle failed calls gracefully.
- No gas-limiting mechanism (try/catch, call.value().gas()) is used.

## ✅ Fixed Code

```solidity

function claim(address[] calldata recipients) external {
    for (uint i = 0; i < recipients.length; i++) {
        if (!claimed[recipients[i]]) {
            claimed[recipients[i]] = true;
            (bool sent, ) = payable(recipients[i]).call{value: 1 ether, gas: 5000}(""); // ✅ safe call
            if (!sent) {
                // optionally log failure
            }
        }
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use low-level call with gas limits instead of transfer or send.
- Handle failed transfers gracefully using pull payment patterns.
- Do not allow untrusted external logic (e.g., looping over arbitrary addresses).

### Additional Safeguards

- Log failed calls and allow users to retry (pull-based withdrawals).
- Use batching to avoid single-point failure in loops.
- Avoid using recipient contracts for sensitive logic unless whitelisted.

### Detection Methods

- Slither: unchecked-low-level-calls, dos-loop, and transfer-usage detectors.
- Manual audit of loop + external call patterns.
- Gas profiling tools (Tenderly, Remix) to simulate edge cases.

## 🕰️ Historical Exploits

- **Name:** Relayer Contract Gas Griefing 
- **Date:** Unspecified 
- **Loss:** Potential denial-of-service (DoS) impact 
- **Post-mortem:** [Link to post-mortem](https://scsfg.io/hackers/griefing/) 


## 📚 Further Reading

- [Insufficient Gas Griefing walkthrough – Bailsec.io](https://bailsec.io/tpost/l9ga6uhe01-insufficient-gas-griefing-walkthrough)
- [Understanding Smart Contract Gas Griefing Attacks – Orochi Network](https://orochi.network/blog/Understanding-Smart-Contract-Gas-Griefing-Attacks) 
- [Secure Gas Forwarding in Smart Contracts: Preventing Insufficient Gas – Varkiwi](https://blog.varkiwi.com/2025/03/13/Check-Forwarded-Gas.html) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Gas Griefing 
severity: M
score:
impact: 3
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Affects availability—can lock out legitimate users from claiming funds.
- **Exploitability**: Attacker only needs a revert/fallback contract.
- **Reachability**: Frequently present in token airdrops, reward distributions.
- **Complexity**: Low to medium—attack requires deploying a smart contract with revert logic.
- **Detectability**: Easily caught by static tools and audit if patterns are known.