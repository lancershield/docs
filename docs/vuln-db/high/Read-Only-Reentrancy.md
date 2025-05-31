# Read-Only Reentrancy

```YAML
id: LS44H
title: Read-Only Reentrancy
baseSeverity: H
category: reentrancy
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: Return value tampering via on-chain call injection
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-841: Improper Enforcement of Behavioral Workflow
swc: SWC-107: Reentrancy
```

## 📝 Description

- Read-only reentrancy is a class of reentrancy vulnerability where attackers exploit a view or pure function's reliance on staticcall to an external contract that returns manipulatable data.
- Although no state is changed during the call, the attacker controls the return value of a staticcall, often by using proxy contracts or reentrant fallbacks. This is particularly dangerous in DeFi protocols when:
- Calculations for rewards, balances, or collateral use external view functions
- External balanceOf() or getReserves() are trusted for token accounting
- Static calls return attacker-controlled values mid-execution
- This enables an attacker to manipulate price feeds, staking balances, or other logic paths without touching state directly, bypassing standard reentrancy guards.

## 🚨 Vulnerable Code

```solidity

function getUserBalance(address user) public view returns (uint256) {
    return stakingContract.balanceOf(user); // ❌ attacker can manipulate this during call
}

function claimReward() external {
    uint256 balance = getUserBalance(msg.sender);
    require(balance > 0, "No stake");
    uint256 reward = balance * 100;
    rewardToken.transfer(msg.sender, reward);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A user deposits into a staking contract that uses balanceOf() to calculate rewards.
2. The attacker uses a contract as their address that implements balanceOf() in a fallback-style manner.
3. When claimReward() calls balanceOf(), the attacker's contract returns a manipulated value (e.g., 1e30) just for that view call.
4. The main contract calculates excessive rewards based on fake balances, and transfers tokens.
5. The staking pool is drained without violating reentrancy guards (as the exploit is read-only).

**Assumptions:**

- Contract depends on external view or staticcall for critical logic.
- No whitelist or contract-type checks are enforced.
- Return values are trusted directly without cryptographic verification or limits.

## ✅ Fixed Code

```solidity

function claimReward() external {
    uint256 balance = userBalances[msg.sender]; // ✅ use internal accounting
    require(balance > 0, "No stake");
    uint256 reward = balance * 100;
    rewardToken.transfer(msg.sender, reward);
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "If external contract allows reentrancy via view calls, this could manipulate system logic significantly."
- context: "DeFi Governance or DAO Voting System"
  severity: C
  reasoning: "Could be used to pass malicious proposals or rig governance outcomes."
- context: "Read-only oracle in a private or whitelisted environment"
  severity: M
  reasoning: "Lower impact if the caller set is trusted and access is controlled."
```

## 🛡️ Prevention

### Primary Defenses

- Avoid trusting return values from external contracts in view/pure logic.
- Rely on internal storage for critical calculations.
- Use immutable or trusted oracles, not externally updatable sources.

### Additional Safeguards

- Use isContract() or interface checking before calling external view functions.
- Cap external return values or apply sanity bounds (e.g., require(balance < X)).

### Detection Methods

- Static analysis for external call, delegatecall, or interface calls inside view functions.
- Tools: Slither (reentrancy-eth, view-call), Echidna, fuzzing with mock reentrant contracts

## 🕰️ Historical Exploits

- **Name:** Cream Finance Price Manipulation 
- **Date:** 2021 
- **Loss:** ~$37.5M 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/cream-rekt/)
  
## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107) 
- [CWE-841: Improper Workflow Enforcement](https://cwe.mitre.org/data/definitions/841.html) 
- [OpenZeppelin Defender – How to Simulate Reentrancy](https://blog.openzeppelin.com/)

--- 

## ✅ Vulnerability Report

```markdown
id: LS44H
title: Read-Only Reentrancy
severity: H
score:
impact: 4  
exploitability: 3  
reachability: 5 
complexity: 2  
detectability: 4  
finalScore: 3.75
```

---

## 📄 Justifications & Analysis

- **Impact**: Can distort on-chain logic without altering state, affecting staking, oracles, rewards, and more.
- **Exploitability**: Relatively easy using fallback contracts that spoof return values.
- **Reachability**: Frequent in protocols that call balanceOf(), reserves(), or similar on external assets.
- **Complexity**: Requires knowledge of fallback patterns, but tools exist to automate the attack.
- **Detectability**: Detectable with code review and Slither reentrancy plugins.
