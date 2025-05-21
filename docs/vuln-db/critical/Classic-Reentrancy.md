# Classic Reentrancy

```YAML
id: TBA
title: Classic Reentrancy
severity: C
category: reentrancy
language: solidity
blockchain: [ethereum]
impact: Full or partial contract balance drain
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-841
swc: SWC-107
```

## 📝 Description

- Classic reentrancy occurs when a smart contract sends Ether to an external contract using a low-level call (e.g., call, send, or transfer) before updating its internal state. 
- If the recipient is a malicious contract, it can recursively call back into the vulnerable function, repeating the withdrawal process before the state is updated, thereby draining funds or bypassing logic constraints.
- This vulnerability was the root cause of the infamous DAO hack in 2016, resulting in over $60M in stolen Ether and the eventual Ethereum hard fork.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract VulnerableVault {
    mapping(address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        // ❌ Send Ether before updating state
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");

        balances[msg.sender] = 0; // State update happens too late
    }
}
```

## 🧪 Exploit Scenario

1. Attacker deposits 1 ETH into VulnerableVault.
2. Attacker calls withdraw(). During the call, their fallback function is triggered.
3. In the fallback, attacker recursively calls withdraw() again.
4. Since balances[msg.sender] has not yet been set to 0, they withdraw again.
5. This repeats until the contract is drained of ETH.

**Assumptions:**

- The recipient is a contract capable of reentry.
- The vulnerable contract uses call, send, or transfer before updating state.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeVault {
    mapping(address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        balances[msg.sender] = 0; // ✅ Update state before sending
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Follow Check-Effects-Interactions pattern.
- Use nonReentrant modifiers via ReentrancyGuard from OpenZeppelin.

### Additional Safeguards

- Minimize use of external calls, especially to unknown or user-provided addresses.
- Avoid using call() unless absolutely necessary; transfer() and send() are safer (pre-2300 gas change).

### Detection Methods

- Static analysis for functions that call external addresses before state mutation.
- Use Slither’s reentrancy-vulnerabilities detector.
- Manual audit of withdrawal and callback flows.

## 🕰️ Historical Exploits
 
- **Name:** Rari Capital Fuse Pool Exploit 
- **Date:** 2022-04-30 
- **Loss:** Over $80 million 
- **Post-mortem:** [Link to post-mortem](https://medium.com/immunefi/the-ultimate-guide-to-reentrancy-19526f105ac)
  
## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107/)
- [Solidity Security Patterns: Checks-Effects-Interactions](https://docs.soliditylang.org/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) 
- [OpenZeppelin ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) 
- [The DAO Incident Timeline](https://ethereum.org/en/history/#the-dao)

---

## ✅ Vulnerability Report 
```markdown
id: TBA
title: Classic Reentrancy
severity: C
score:
impact: 5 
exploitability: 5 
reachability: 5 
complexity: 2  
detectability: 4  
finalScore: 4.6
```

---

## 📄 Justifications & Analysis

- **Impact**: Contract funds can be completely drained by a single attacker.
- **Exploitability**: Straightforward with a malicious contract and a vulnerable flow.
- **Reachability**: Found in many legacy contracts and newer unguarded withdrawal logic.
- **Complexity**: Moderate; attacker just needs fallback function and balance.
- **Detectability**: High—readily caught by tools and basic audit checklists.