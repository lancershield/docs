# Force-Feeding Attacks

```YAML
id: TBA
title: Force-Feeding Attacks 
baseSeverity: M
category: ether-injection
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: Inconsistent ETH accounting, balance-based logic bypass
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-668
swc: SWC-132
```

## 📝 Description

- Force-feeding attacks occur when an attacker sends ETH directly to a contract address using methods that bypass fallback or receive functions, such as:
- Using selfdestruct(address) to force-send ETH
- Direct transfers from coinbase or contracts without calling functions
- This ETH cannot be refused and does not invoke logic, but it increases the contract’s balance. 
- If the contract includes balance-sensitive logic (e.g., requiring address(this).balance == 0 or gating withdrawals on contract balance).

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract Vault {
    address public owner;

    constructor() payable {
        owner = msg.sender;
    }

    function claimRefund() external {
        require(address(this).balance == 0, "Vault has ETH"); // ❌ susceptible
        // Allow user to claim back collateral only if vault is empty
        payable(msg.sender).transfer(1 ether);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A smart contract Vault has a condition like require(address(this).balance == 0) in a refund or permission check.
2. An attacker deploys a small contract that holds 1 wei and calls selfdestruct(vaultAddress).
3. This forces ETH into the Vault contract without calling any function, bypassing access control.
4. The Vault's logic that depends on exact balance assumptions now fails — e.g., refund functions are permanently disabled or misbehave.
5. Users are unable to proceed, and the contract becomes stuck or manipulated based on the forced ETH presence.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeVault {
    address public owner;

    constructor() payable {
        owner = msg.sender;
    }

    function claimRefund(uint256 expectedBalance) external {
        // ✅ Only rely on tracked internal accounting
        require(expectedBalance == 0, "Collateral not cleared");
        payable(msg.sender).transfer(1 ether);
    }
}
```
## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Causes logic inconsistencies but no direct loss unless logic fails critically."
- context: "DeFi pool using `address(this).balance`"
  severity: H
  reasoning: "May result in incorrect rewards, loss of funds, or manipulation."
- context: "Donation address with no logic tied to balance"
  severity: L
  reasoning: "No harm if logic does not rely on contract balance."
```

## 🛡️ Prevention

### Primary Defenses

- Never use address(this).balance as the sole source of truth
- Track ETH via internal variables (e.g., deposits[msg.sender])

### Additional Safeguards

- Add an owner-only sweep function for unexpected ETH
- Emit events when funds are received unexpectedly

### Detection Methods

- Static analysis for balance-dependent conditions
- Fuzzing tests using selfdestruct() ETH delivery patterns
- Tools: Slither (ether-transfer, balance-dependence), Echidna, Foundry invariant tests

## 🕰️ Historical Exploits

- **Name:** Parity Wallet Refund Lock 
- **Date:** 2017-11 
- **Loss:** ~$150M  
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert.html)  
- **Name:** Gnosis Safe Misunderstood Refund Logic 
- **Date:** 2021-12 
- **Loss:** N/A (refund behavior broken due to unexpected ETH presence) 
- **Post-mortem:** [Link to post-mortem](https://forum.gnosis.io) 
  
## 📚 Further Reading

- [SWC-132: Unexpected Ether Balance](https://swcregistry.io/docs/SWC-132) 
- [CWE-668: Exposure of Resource to Wrong Sphere](https://cwe.mitre.org/data/definitions/668.html) 
- [Solidity Security Considerations – Ether Injection](https://docs.soliditylang.org/en/latest/security-considerations.html#ether-injection)  
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Force-Feeding Attacks 
severity: M
score:
impact: 4    
exploitability: 4 
reachability: 3  
complexity: 2     
detectability: 4  
finalScore: 3.65
```

---

## 📄 Justifications & Analysis

- **Impact**: Can lock critical user flows or block contract recovery
- **Exploitability**: Easy to trigger via standard selfdestruct() mechanics
- **Reachability**: Common in DeFi refund logic and ETH-based gating
- **Complexity**: Requires understanding of low-level ETH transfer mechanics
- **Detectability**: Detectable with Slither and careful balance logic review