# Reentrancy Vulnerability

```YAML

id: TBA
title: Reentrancy on External Contract Calls
severity: H
category: reentrancy
language: solidity
blockchain: [ethereum]
impact: Repeated unauthorized state execution
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-841
swc: SWC-107
```

## 📝 Description

- Reentrancy is a critical vulnerability where an external call is made before internal state changes are finalized, allowing malicious contracts to re-enter the vulnerable function repeatedly. 
- This can lead to double-withdrawals, drained funds, or broken invariants. 
- It is most common in fallback functions or when using `call`, `send`, or `transfer` without proper access control and state ordering.

## 🚨 Vulnerable Code

```solidity
contract ReentrancyVuln {
    mapping(address => uint256) public balances;

    function withdraw() public {
        require(balances[msg.sender] > 0, "No balance");
        payable(msg.sender).call{value: balances[msg.sender]}(""); // reentrancy point
        balances[msg.sender] = 0; // state change after external call
    }

    receive() external payable {
        balances[msg.sender] += msg.value;
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker deposits some ETH.
2. Attacker’s fallback contract calls withdraw() and during call{value: ...}, re-enters withdraw() again.
3. Since balances[msg.sender] has not been zeroed, attacker withdraws repeatedly.
4. Contract balance is drained before state is updated.

**Assumptions:**

- No reentrancyGuard or Checks-Effects-Interactions pattern.
- Attacker uses a malicious contract with a fallback function.

## ✅ Fixed Code

```solidity

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SafeWithdraw is ReentrancyGuard {
    mapping(address => uint256) public balances;

    function withdraw() public nonReentrant {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "No balance");
        balances[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }

    receive() external payable {
        balances[msg.sender] += msg.value;
    }
}

```

## 🛡️ Prevention

### Primary Defenses

- Use Checks-Effects-Interactions pattern: update state before external calls.
- Use ReentrancyGuard modifier from OpenZeppelin.
- Avoid calling external contracts unless strictly necessary.

### Additional Safeguards

- Use transfer() or send() for fixed-gas payouts, if compatible.
- If using call(), wrap with nonReentrant and restrict fallbacks.
- Monitor unusual recursive call patterns in logs.

### Detection Methods

- Slither: reentrancy-eth detector.
- Mythril: symbolic path analysis for reentrant logic.
- Manual inspection of call, delegatecall, or fallback flows.

## 🕰️ Historical Exploits

- **Name:** The DAO Hack
- **Date:** 2016-06-17 
- **Loss:** ~$60M 
- **Post-mortem:** [Link to post-mortem](https://blog.slock.it/the-dao-hack-explained-62429dbabf62) -
  
  
- **Name:** dForce/Lendf.Me Reentrancy Attack 
- **Date:** 2020-04-19 
- **Loss:** ~$25M
- **Post-mortem:** [Link to post-mortem](https://medium.com/dforcenetofficial-announcement-regarding-lendf-me-incident-18c8995e4f17) 


## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107) 
- [OpenZeppelin Docs: ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) 
- [Trail of Bits: Understanding Reentrancy](https://blog.trailofbits.com/2022/07/18/reentrancy-illustrated/) 


```markdown
id: TBA
title: Reentrancy on External Contract Calls
severity: H
score:
impact: 5         
exploitability: 5 
reachability: 4   
complexity: 3     
detectability: 3  
finalScore: 4.5
```

---

## 📄 Justifications & Analysis

- **Impact**: A successful exploit can fully drain contract funds.
- **Exploitability**: Publicly known, simple to exploit if protections are absent.
- **Reachability**: Withdrawals, payments, and callbacks often expose this.
- **Complexity**: Requires attacker contract with fallback logic.
- **Detectability**: Tools detect reentrancy, but may miss second-order paths.
