# Gas Limit Vulnerabilities

```YAML
id: TBA
title: Gas Limit Vulnerabilities in Unbounded Loops 
severity: H
category: gas-limit
language: solidity
blockchain: [ethereum]
impact: Denial of service or function inaccessibility
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-400
swc: SWC-128
```

## 📝 Description

- Gas limit vulnerabilities arise when functions can consume excessive gas due to unbounded operations like looping over dynamic storage arrays or performing too many writes. When the gas required to execute exceeds the block gas limit, transactions revert and critical functions become unusable. 
- This leads to denial of service, especially in `withdraw`, `distribute`, or `claim` functions affecting user funds.

## 🚨 Vulnerable Code

```solidity
address[] public recipients;

function distribute() external {
    uint256 share = address(this).balance / recipients.length;
    for (uint256 i = 0; i < recipients.length; i++) {
        payable(recipients[i]).transfer(share); // can exceed gas limit if many recipients
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Over time, the list of recipients grows large (e.g., 1000+ addresses).
2. When distribute() is called, the loop tries to process every address.
3. Execution exceeds the block gas limit → transaction reverts.
4. Funds become locked; no one can trigger the distribution anymore.

**Assumptions:**

- Contract allows unlimited growth of dynamic arrays.
- No batching or pagination logic present in looping function.

## ✅ Fixed Code

```solidity

function distributeBatch(uint256 start, uint256 end) external {
    require(end <= recipients.length, "Out of range");
    uint256 share = address(this).balance / recipients.length;
    for (uint256 i = start; i < end; i++) {
        payable(recipients[i]).transfer(share);
    }
}

```

## 🛡️ Prevention

### Primary Defenses

- Avoid unbounded loops over storage in a single transaction.
- Implement batching or pagination (start–end index) for processing.
- Consider using pull-based models instead of push-based distributions.

### Additional Safeguards

- Add gas guards or gasleft() checks for fail-safe exits.
- Limit the maximum array size or queue length.
- Use off-chain orchestration (e.g., relayers, automation bots).

### Detection Methods

- Slither: unbounded-loop, dos, gas-limit detectors.
- Hardhat/Ganache: simulate long-running loops with large inputs.
- Fuzzing and load testing with long lists or high iteration scenarios.

## 🕰️ Historical Exploits

- **Name:** Fomo3D Jackpot Finalization Blocked 
- **Date:** 2018 
- **Loss:** Jackpot stuck due to exceeding gas limit 
- **Post-mortem:** [Link to post-mortem](https://www.reddit.com/r/ethtrader/comments/94y5ry/fomo3d_gas_limit_bug/)
- **Name:** Fomo3D Block Stuffing Attack 
- **Date:** 2018-08-22 
- **Loss:** Jackpot manipulation through gas limit exploitation 
- **Post-mortem:** [Link to post-mortem](https://medium.com/@0xkaden/the-encyclopedia-of-smart-contract-attacks-vulnerabilities-dfc1129fdaac) 

## 📚 Further Reading

- [SWC-113: DoS with Block Gas Limit – SWC Registry](https://swcregistry.io/docs/SWC-113/) 
- [Smart Contract Vulnerabilities Unveiled: Block Gas Limit Vulnerability](https://medium.com/coinmonks/smart-contract-vulnerabilities-unveiled-block-gas-limit-vulnerability-6499fd5c579b) 
- [Smart Contract Security Risks: Today's 10 Top Vulnerabilities – Cobalt](https://www.cobalt.io/blog/smart-contract-security-risks) 

---
## ✅ Vulnerability Report 

```markdown
id: TBA
title: Gas Limit Vulnerabilities 
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Critical functions like withdrawals and distributions become unreachable.
- **Exploitability**: Any user or attacker can trigger it once state grows large enough.
- **Reachability**: Most protocols expose publicly callable loops in payouts or claims.
- **Complexity**: Low effort; only requires invoking function with bloated internal data.
- **Detectability**: Static analyzers and gas profilers catch this with simulated inputs.
