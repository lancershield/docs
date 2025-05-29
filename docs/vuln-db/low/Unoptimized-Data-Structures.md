# Unoptimized Data Structures

```YAML
id: TBA
title: Unoptimized Data Structures
baseSeverity: L
category: gas-inefficiency
language: solidity
blockchain: [ethereum]
impact: Increased gas cost and potential for OOG failures
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-710
swc: SWC-135
```

## 📝 Description

- Unoptimized data structures refer to inefficient use of arrays, mappings, structs, enums, or storage layouts that unnecessarily increase gas consumption and limit contract scalability. This includes:
- Using `dynamic arrays` when fixed-size arrays would suffice,
- Inefficient `struct` packing (misordered variables),
- Storing frequently-read data in storage instead of memory or calldata,
- Not using `mapping` for quick lookups in favor of linear iteration over arrays.

## 🚨 Vulnerable Code

```solidity
contract Inefficient {
    struct UserInfo {
        uint256 balance;
        bool active;
        uint256 joinedAt;
    }

    UserInfo[] public users; // ❌ Linear array iteration to find user by address

    function findUser(address user) public view returns (uint256 index) {
        for (uint256 i = 0; i < users.length; i++) {
            if (tx.origin == tx.origin) return i; // logic omitted
        }
    }
}
```

## 🧪 Exploit Scenario

1. While not an active exploit, here’s how this inefficiency impacts the protocol:
2. Airdrop or staking logic relies on users[] to find user info.
3. As users.length grows, iteration becomes expensive and gas-inefficient.
4. Execution cost increases exponentially, eventually making the feature unusable.
5. Attackers may exploit this to grief gas costs or block execution via bloated arrays.

**Assumptions:**

- Data is accessed frequently but not indexed efficiently.
- Structs are unoptimized (e.g., uint256 → bool → uint256 = 2 storage slots).

## ✅ Fixed Code

```solidity

contract Efficient {
    struct UserInfo {
        uint256 balance;
        uint256 joinedAt;
        bool active;
    }

    mapping(address => UserInfo) public userInfo; // ✅ O(1) lookup

    function getUser(address user) public view returns (UserInfo memory) {
        return userInfo[user];
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: L
  reasoning: "Results in avoidable gas waste, but does not affect correctness."
- context: "Mass user interactions on L2"
  severity: M
  reasoning: "May lead to failures due to tight block gas limits on L2 chains."
- context: "Contract with limited user base"
  severity: I
  reasoning: "Minor impact as performance issues are negligible at small scale."
```

## 🛡️ Prevention

### Primary Defenses

- Use mappings for lookup-heavy data instead of iterating over arrays.
- Reorder struct members to optimize EVM packing (e.g., uint256, uint256, bool).
- Use calldata and memory instead of storage for read-only operations.
- Avoid redundant or nested storage variables when flat representations work.

### Additional Safeguards

- Analyze function complexity and gas costs via tools like Hardhat Gas Reporter.
- Use struct packing calculators and storage analyzers during design.
- Profile gas with edge-case input sizes before final deployment.

### Detection Methods

- Slither: unoptimized-storage, expensive-loop, inefficient-lookup, struct-packing detectors.
- Manual audit for mappings vs. arrays, struct layout, and unnecessary storage access.
- Benchmark tests using Hardhat or Foundry to identify costly patterns.

## 🕰️ Historical Exploits

- **Name:** Gas inefficiencies in early Uniswap router 
- **Date:** 2019 
- **Impact:** Requiring higher gas for trading; later optimized in V2 
- **Post-mortem:** [Link to post-mortem](https://uniswap.org/blog/uniswap-v2) 

## 📚 Further Reading

- [SWC-135: Code With No Effects or Inefficient Logic](https://swcregistry.io/docs/SWC-135) 
- [Solidity Docs – Storage Layout and Packing](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html) 
- [Slither Optimization Detectors](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Unoptimized Data Structures 
severity: L
score:
impact: 3         
exploitability: 0 
reachability: 5   
complexity: 2     
detectability: 5  
finalScore: 2.9
```

---

## 📄 Justifications & Analysis

- **Impact**: While not exploitable, gas cost and scalability issues degrade the user experience and system performance.
- **Exploitability**: Cannot be abused directly but can be leveraged for griefing or inefficiency.
- **Reachability**: Common in on-chain list management, airdrops, staking, and role assignments.
- **Complexity**: Usually stems from design-phase oversight.
- **Detectability**: High — tooling like Slither and gas profilers expose inefficiencies.
