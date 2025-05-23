# ReentrancyGuard

```YAML
id: TBA
title: ReentrancyGuard 
severity: G
category: reentrancy
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: Double withdrawals, state corruption, protocol drain
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-841
swc: SWC-107
```

## 📝 Description

- Smart contracts that include external calls (such as transfer, call, or token hooks) within state-changing functions are susceptible to reentrancy if they do not use appropriate locking mechanisms like ReentrancyGuard.
- ReentrancyGuard is an OpenZeppelin-provided pattern that ensures a function cannot be re-entered during execution. If omitted or misused:
- Attackers can reenter a contract mid-function, before state is finalized.
- This enables double withdrawals, reward duplication, vault draining, or arbitrary state corruption.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Vault {
    mapping(address => uint256) public balances;

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        (bool success, ) = msg.sender.call{value: amount}(""); // ❌ reentrant opportunity
        require(success, "Transfer failed");

        balances[msg.sender] = 0; // ❌ state updated after external call
    }

    receive() external payable {}
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker deposits 1 ETH into the vulnerable vault.
2. They deploy a malicious contract with a fallback receive() that calls withdraw() again.
3. When the first withdraw() sends ETH, the fallback triggers another withdraw().
4. Since balances[msg.sender] is not yet zeroed, the attacker drains multiple ETH.

**Assumptions:**

- External call made before internal state is finalized.
- No reentrancy guard or modifier is used.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SafeVault is ReentrancyGuard {
    mapping(address => uint256) public balances;

    function withdraw() external nonReentrant {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        balances[msg.sender] = 0; // ✅ update state before interaction

        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }

    receive() external payable {}
}
```

## 🛡️ Prevention

### Primary Defenses

- Always use ReentrancyGuard for withdrawal and claim functions
- Apply nonReentrant modifier on external/public functions interacting with untrusted targets

### Additional Safeguards

- Avoid using call{value:...} unless necessary
- Use pull-payment patterns (user claims instead of being pushed)

### Detection Methods

- Audit for external calls followed by state writes
- Tools: Slither (reentrancy-vulnerabilities, calls-before-state-updates), MythX, Foundry fuzzing

## 🕰️ Historical Exploits

- **Name:** dForce Lendf.Me Exploit 
- **Date:** 2020-04 
- **Loss:** ~$25 million drained via ERC-777 reentrancy
- **Post-mortem:** [Link to post-mortem](https://dforce.network/blog/post-mortem-analysis-of-lendfme-incident)
  
## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107/)
- [OpenZeppelin – ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) 
- [The DAO Hack Explained – ConsenSys](https://media.consensys.net/the-dao-hack-explained-8abb40e12795)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: ReentrancyGuard 
severity: G
score:
impact: 5    
exploitability: 4 
reachability: 5  
complexity: 2    
detectability: 5  
finalScore: 4.4
```

---

## 📄 Justifications & Analysis

- **Impact**: Funds can be drained or corrupted via reentrancy
- **Exploitability**: Simple for attackers with fallback-enabled contracts
- **Reachability**: Found in any call() or token interaction logic
- **Complexity**: Low; even basic Solidity attackers can abuse
- **Detectability**: Easily flagged by tooling or audit heuristics