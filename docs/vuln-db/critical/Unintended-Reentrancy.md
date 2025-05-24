# Unintended Reentrancy

```YAML
id: TBA
title: Unintended Reentrancy
severity: C
category: reentrancy
language: solidity
blockchain: [ethereum, bsc, polygon, arbitrum, optimism]
impact: Unauthorized repeated execution of sensitive logic
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-841
swc: SWC-107
```

## 📝 Description

- Unintended Reentrancy occurs when a smart contract makes an external call before internal state is fully updated, allowing the callee to re-enter and re-trigger the same function in an inconsistent state.
- This vulnerability:
- Is often missed when developers assume function logic is atomic
- May occur even in functions not explicitly marked payable
- Can affect access control, balances, or governance flows if callbacks are allowed
- Unlike “expected” reentrancy (like in receive() functions), this flaw is “unintended” when any external call allows a contract to exploit reentrant control flow.

## 🚨 Vulnerable Code

```solidity

mapping(address => uint256) public balances;

function withdraw(uint256 amount) external {
    require(balances[msg.sender] >= amount, "Insufficient");
    (bool sent, ) = msg.sender.call{value: amount}(""); // ❌ external call
    require(sent, "Failed");
    balances[msg.sender] -= amount; // ❌ updated after transfer
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker deposits ETH into the vulnerable contract.
2. Calls withdraw(amount) which invokes msg.sender.call{value: amount}.
3. Attacker’s contract has a fallback function that re-enters withdraw() again.
4. Because balances[msg.sender] is not yet updated, the attacker can call withdraw() multiple times.
5. Contract balance is drained before the internal balance is updated.



## ✅ Fixed Code

```solidity

function withdraw(uint256 amount) external {
    require(balances[msg.sender] >= amount, "Insufficient");
    balances[msg.sender] -= amount; // ✅ update before external call
    (bool sent, ) = msg.sender.call{value: amount}("");
    require(sent, "Failed");
}
```

## 🛡️ Prevention

### Primary Defenses

- Follow checks-effects-interactions pattern
- Use ReentrancyGuard modifiers on all state-changing external functions
- Avoid calling untrusted external contracts directly

### Additional Safeguards

- Use pull payments instead of push (users withdraw funds manually)
- Restrict access to critical functions during execution (mutex pattern)
- Write invariant-based tests to simulate recursive logic

### Detection Methods

- Static analysis for call, delegatecall, send, or transfer followed by state updates
- Fuzzing for repeated external callback attempts
- Tools: Slither (reentrancy-eth, reentrancy-no-guard), MythX, Foundry tests

## 🕰️ Historical Exploits

- **Name**: Curve Finance Reentrancy Exploit 
- **Date**: 2023-07-30 
- **Loss**: ~$70M 
- **Post-mortem**: [Link to post-mortem](https://www.trustbytes.io/blog/reentrancy-attacks)

## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107)
- [CWE-841: Improper Enforcement of Behavioral Workflow](https://cwe.mitre.org/data/definitions/841.html) 
- [OpenZeppelin ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) 
- [Solidity Security Patterns – Checks-Effects-Interactions](https://docs.soliditylang.org/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) 


## ✅ Vulnerability Report

```markdown
id: TBA
title: Unintended Reentrancy
severity: C
score:
impact: 5 
exploitability: 4 
reachability: 5 
complexity: 3  
detectability: 4  
finalScore: 4.45
```

---

## 📄 Justifications & Analysis

- **Impact**: Can fully drain contract ETH or tokens; major loss in live cases like The DAO.
- **Exploitability**: Accessible to any attacker with a smart contract and enough gas.
- **Reachability**: Common in withdrawal, payout, and ERC721 safeTransferFrom flows.
- **Complexity**: Medium—requires reentrant logic knowledge, but easy to script.
- **Detectability**: Easily detected with static tools or audit attention to call patterns.

