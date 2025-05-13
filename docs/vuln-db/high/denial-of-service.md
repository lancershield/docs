
# Denial of Service

```YAML
id: TBA
title: Denial of Service via Unbounded Loop 
severity: H
category: denial-of-service
language: solidity
blockchain: [ethereum]
impact: Contract becomes unusable or stuck
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-400
swc: SWC-113
```

## 📝 Description

- Denial of Service (DoS) in smart contracts occurs when an attacker can prevent legitimate users from interacting with a contract, either temporarily or permanently. 
- This can be achieved by exploiting resource-intensive operations (e.g., unbounded loops), failing external calls, unexpected reverts, or storage manipulation that causes subsequent operations to fail.

- In Ethereum, since each transaction has a gas limit, a common DoS vector is introducing logic that exceeds the block gas limit or causes reversion due to invalid assumptions (like looping through a growing array).

## 🚨 Vulnerable Code

```solidity
mapping(address => uint[]) public userDeposits;

function withdrawAll() external {
    uint[] storage deposits = userDeposits[msg.sender];
    for (uint i = 0; i < deposits.length; i++) {
        _processWithdrawal(deposits[i]); // Assume this costs significant gas
    }
    delete userDeposits[msg.sender];
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker makes hundreds or thousands ...[msg.sender].

2. Calls withdrawAll() which loops over all entries...

3. The loop execution exceeds the block gas limit → reverts.

4. The attacker is now permanently locked out... 

**Assumptions:**

- The contract allows unlimited or unbounded entries into arrays or mappings.

- There is no gas-bound safety mechanism in looping constructs.

## ✅ Fixed Code

```solidity
function withdrawBatch(uint start, uint end) external {
    uint[] storage deposits = userDeposits[msg.sender];
    require(end <= deposits.length, "Invalid end index");

    for (uint i = start; i < end; i++) {
        _processWithdrawal(deposits[i]);
    }

    if (end == deposits.length) {
        delete userDeposits[msg.sender];
    }
}
```

### 🛡️ Prevention

### Primary Defenses

- Avoid unbounded loops in external/public functions.
- Introduce batching mechanisms (e.g., start and end indices).
- Set gas guards and enforce maximum data sizes where feasible.

### Additional Safeguards

- Implement rate limiting or per-call caps.
- Use gas estimation during testing to simulate worst-case  scenarios.

### Detection Methods

- Detect via static analysis tools (e.g., Slither: unbounded-loop, dos detectors).
- Fuzzing and gas profiling during QA.
- Formal verification of loop safety and invariants.

## 🕰️ Historical Exploits

- GovernMental (2016): Attackers used large data insertion to block contract execution.
- Parity Multisig (2017): Vulnerability in library handling resulted in contract lockout.
- SpankChain (2018): DoS via unexpected reverts in fallback calls.

-  **Name:** GovernMental DoS Vulnerability  
-  **Date:** 2016-06-11  
-  **Loss:** N/A  
-  **Post-mortem:** [Link to post-mortem](https://www.reddit.com/r/ethereum/comments/4np972/governmental_dapp_scam_or_honeypot/)



## 📚 Further Reading

- SWC-113: DoS with Block Gas Limit

- Ethereum Smart Contract Best Practices – Denial of Service

- Slither Docs – Detecting DoS


- [SWC-113: DoS with Block Gas Limit](https://swcregistry.io/docs/SWC-113)
- [Ethereum Smart Contract Best Practices – Denial of Service](https://consensys.github.io/smart-contract-best-practices/attacks/denial-of-service/)
- [Slither Docs – Detecting DoS](https://github.com/crytic/slither)


---

## ✅ Vulnerability Report 

```
id: vuln__denial_of_service
title: Denial of Service
severity: H
score:
impact: <5>        
exploitability: <4> 
reachability: <5>   
complexity: <1>     
detectability: <3>  
finalScore: <4.3>
```

  ---

## 📄 Justifications & Analysis

- **Impact**: A well-crafted DoS can freeze user funds or key contract functionality, leading to major trust and financial loss.
- **Exploitability**: The attack can be triggered externally with minimal effort (e.g., looping, reverting).
- **Reachability**: Public/external functions without restrictions can be exploited at will.
- **Complexity**: Attack only requires interacting with the contract normally (e.g., through deposits or submissions).
- **Detectability**: While many tools detect unbounded loops, nuanced cases (e.g., user-sourced data) may be overlooked without manual review.