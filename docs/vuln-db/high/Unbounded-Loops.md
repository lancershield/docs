# Unbounded Loops

```YAML
id: TBA
title: Unbounded Loops
severity: H
category: gas
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Out-of-gas errors, DoS for users, blocked withdrawals or reward claims
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-834: Excessive Iteration
swc: SWC-128: DoS With Block Gas Limit
```

## 📝 Description

- Unbounded loops occur when a smart contract executes a for or while loop over a dynamically-sized data structure (e.g., array.length, mapping, or user-set list) without limiting the number of iterations. On blockchains with gas limits, this can lead to:
- Out-of-gas (OOG) errors
- Permanent DoS for functions like withdraw() or claimRewards()
- System-wide failure to execute core logic

## 🚨 Vulnerable Code

```solidity

address[] public stakers;

function distributeRewards() external onlyOwner {
    for (uint256 i = 0; i < stakers.length; i++) {
        rewards[stakers[i]] += 100;
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A staking contract records all participants in a growing address[] stakers array.
2. distributeRewards() is designed to reward all users in one loop.
3. The platform gains popularity; stakers grow to 5,000+ addresses.
4. When the owner calls distributeRewards(), it fails due to gas exhaustion.
5. No rewards are distributed, and the system is effectively locked.

**Assumptions:**

- The contract uses loops over dynamic arrays or mappings without upper bounds.
- The loop logic affects critical functionality such as withdrawals, claims, or updates.

## ✅ Fixed Code

```solidity

function distributeRewardsBatch(uint256 start, uint256 end) external onlyOwner {
    require(end <= stakers.length, "Invalid range");
    for (uint256 i = start; i < end; i++) {
        rewards[stakers[i]] += 100;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid writing loops over user-controlled dynamic data.
- Use pull-based patterns instead of global state mutation.
- Introduce batching, limits, or checkpoints to control loop size.

### Additional Safeguards

- Include require(gasleft() > X) logic if nearing gas-critical operations.
- Use off-chain scripts or subgraphs to process bulk logic safely.
- Simulate max loop sizes during unit testing.

### Detection Methods

- Slither: unbounded-loop, calls-loop-array-length
- Manual review of loops over array.length, mapping keys, etc.
- Fuzzing with large inputs to observe gas ceiling

## 🕰️ Historical Exploits
 
- **Name:** Synthetix Reward Debt Loop 
- **Date:** 2020 
- **Loss:** ~$30,000 
- **Post-mortem:** [Link to post-mortem](https://github.com/Synthetixio) 

📚 Further Reading

- [SWC-128: DoS with Block Gas Limit](https://swcregistry.io/docs/SWC-128) 
- [CWE-834: Excessive Iteration](https://cwe.mitre.org/data/definitions/834.html) 
- [Solidity Security Considerations – Loops](https://docs.soliditylang.org/en/latest/security-considerations.html#gas-limit-and-loops)
- [OpenZeppelin – Pull Payment Pattern](https://docs.openzeppelin.com/contracts/4.x/api/security#PullPayment) 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Unbounded Loops
severity: H
score:
impact: 4  
exploitability: 3  
reachability: 4   
complexity: 2   
detectability: 4  
finalScore: 3.65
```

---

## 📄 Justifications & Analysis

- **Impact**: Funds become stuck or actions fail due to OOG errors in core logic.
- **Exploitability**: Anyone can increase data size (e.g., via deposits) and trigger failure.
- **Reachability**: Common in farming, staking, and governance contracts.
- **Complexity**: Basic design flaw; doesn’t require technical skill to abuse.
- **Detectability**: Static tools like Slither flag these reliably.
