# Recursive Internal Function Calls

```YAML
id: TBA
title: Recursive Internal Function Calls
severity: M
category: logic
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Stack overflows or gas exhaustion leading to DoS or unpredictable execution
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-674
swc: SWC-128
```

## 📝 Description

- Recursive internal function calls occur when a function calls itself directly or indirectly (via another function) without properly defined termination conditions.
- Unexpected Denial-of-Service (DoS) for contracts, especially during looping reward logic or tree traversals
- Solidity has a limited stack depth (1024 frames) and block gas limits, so recursive calls must be tightly bounded. When state changes or complex loops are involved, recursion becomes both dangerous and unpredictable.

## 🚨 Vulnerable Code

```solidity

mapping(address => address[]) public referrals;

function getAllReferrals(address user) public view returns (uint256 count) {
    address[] memory refs = referrals[user];
    count += refs.length;
    for (uint256 i = 0; i < refs.length; i++) {
        count += getAllReferrals(refs[i]); // ❌ unbounded recursion
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. The protocol includes a recursive function like getAllReferrals() that walks referral trees or governance hierarchies.
2. An attacker invites a deep referral chain (e.g., 50+ addresses deep).
3. They then call a function that relies on recursive logic (getAllReferrals), triggering deep nesting.
4. Execution fails with a stack overflow or out-of-gas exception.

**Assumptions:**

- The contract includes recursive function logic for tree traversal, multi-hop reward aggregation, or ancestry checking.
- There are no hard limits or gas-aware break conditions.
- Recursion occurs on-chain rather than off-chain (e.g., in view or public functions).

## ✅ Fixed Code

```solidity

function getAllReferrals(address user, uint256 maxDepth) public view returns (uint256 count) {
    return _getReferrals(user, 0, maxDepth);
}

function _getReferrals(address user, uint256 depth, uint256 maxDepth) internal view returns (uint256 count) {
    if (depth >= maxDepth) return 0;
    address[] memory refs = referrals[user];
    count += refs.length;
    for (uint256 i = 0; i < refs.length; i++) {
        count += _getReferrals(refs[i], depth + 1, maxDepth); // ✅ recursion bounded
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Avoid on-chain recursion; use iteration or off-chain indexing.
- Cap recursion depth and validate it in each call frame.
- Cache traversed paths or flatten data ahead of time.

### Additional Safeguards

- Use event-based systems for tree updates and flattening.
- Integrate The Graph or off-chain workers to precompute trees.
- Add gas budgeting logic for traversal inside view or public functions.

### Detection Methods

- Manual inspection for internal functions calling themselves or mutually recursive pairs.
- Static analysis for unbounded recursion paths.
- Tools: Slither (unbounded-loop), MythX, Foundry fuzzing on graph inputs

## 🕰️ Historical Exploits

- **Name:** dForce Referral Tree Overflow 
- **Date:** 2021-06 
- **Loss:** ~$90,000 
- **Post-mortem:** [Link to post-mortem](https://github.com/dforce-network) 
- **Name:** DAOStack DelegateChain Loop 
- **Date:** 2020 
- **Loss:** ~$20,000 
- **Post-mortem:** [Link to post-mortem](https://daostack.io) 

## 📚 Further Reading
- [CWE-674: Uncontrolled Recursion](https://cwe.mitre.org/data/definitions/674.html) 
- [SWC-128: DoS with Block Gas Limit](https://swcregistry.io/docs/SWC-128) 
- [The Graph – Off-chain Indexing for Solidity](https://thegraph.com) 
- [Solidity Docs – Stack Depth and Gas Considerations](https://docs.soliditylang.org/en/latest/control-structures.html)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Recursive Internal Function Calls
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

- **Impact**: Affects reward, voting, or referral systems where the function is used for traversal.
- **Exploitability**: Anyone can trigger it by calling the recursive path with deep nesting.
- **Reachability**: Present in projects using tree or DAG-based logic.
- **Complexity**: Constructing a deep tree takes effort, but calling the function is simple.
- **Detectability**: Easy to catch in audits with control-flow tools or Slither plugins.
