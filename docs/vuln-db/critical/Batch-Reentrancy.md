# Batch Reentrancy with Incomplete State

```YAML
id: TBA
title: Batch Reentrancy with Incomplete State 
severity: C
category: reentrancy
language: solidity
blockchain: [ethereum]
impact: Double execution, drained balances, or unauthorized access
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-841
swc: SWC-107
```

## 📝 Description

- Batch processing functions that loop over user operations (e.g., batchWithdraw(), claimAll(), multiTransfer()) can be vulnerable to reentrancy attacks if state updates are delayed until after external calls or finalized only at the end of the loop.
- In such cases, attackers can re-enter the function before the loop finishes or before critical state changes are applied, enabling:
- Double withdrawals
- Bypassing accounting updates
- Re-entering later iterations of the batch before the loop completes
- This class of vulnerability is often missed because the logic seems sequential, but due to external calls during iteration, malicious contracts can recursively re-trigger the batch logic while internal state remains unfinalized.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract BatchPayout {
    mapping(address => uint256) public balances;

    function claimAll(address[] calldata recipients) external {
        for (uint256 i = 0; i < recipients.length; ++i) {
            address to = recipients[i];
            uint256 amount = balances[to];

            if (amount > 0) {
                (bool sent, ) = to.call{value: amount}(""); // ❌ external call before zeroing balance
                require(sent, "Transfer failed");

                balances[to] = 0; // ❌ update happens after transfer
            }
        }
    }

    receive() external payable {}
}
```

## 🧪 Exploit Scenario

1. Alice’s contract is a recipient in claimAll().
2. Her balance is 10 ETH. She calls claimAll([Alice]).
3. During the call{value: amount}(""), Alice’s contract’s receive() function reenters and calls claimAll() again.
4. Since her balance is not yet zero, the second call succeeds and sends 10 ETH again.
5. Alice drains the vault or claims funds twice.

**Assumptions:**

- External calls are made before final state updates inside batch loops.
- No reentrancy protection (e.g., nonReentrant) is applied.
- A malicious recipient can trigger recursive execution.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SafeBatchPayout is ReentrancyGuard {
    mapping(address => uint256) public balances;

    function claimAll(address[] calldata recipients) external nonReentrant {
        for (uint256 i = 0; i < recipients.length; ++i) {
            address to = recipients[i];
            uint256 amount = balances[to];

            if (amount > 0) {
                balances[to] = 0; // ✅ update state before transfer
                (bool sent, ) = to.call{value: amount}("");
                require(sent, "Transfer failed");
            }
        }
    }

    receive() external payable {}
}
```

## 🛡️ Prevention

### Primary Defenses

- Always update state (e.g., zero balances) before external transfers.
- Use nonReentrant modifier for all batch processing functions.

### Additional Safeguards

- Avoid using call{value: ...}("") unless absolutely necessary.
- In critical protocols, consider pull-based payments instead of batching payouts.

### Detection Methods

- Review for external calls inside loops without prior state finalization.
- Tools: Slither (reentrancy-no-eth, calls-before-state-update), MythX

## 🕰️ Historical Exploits

- **Name:** Hypercerts ERC1155 Batch Mint Reentrancy 
- **Date:** 2023-04 
- **Loss:** ~$120,000  
- **Post-mortem:** [Link to post-mortem](https://www.cyfrin.io/blog/what-is-a-reentrancy-attack-solidity-smart-contracts) 
- **Name:** Sereum Ethereum Reentrancy Study 
- **Date:** 2018-12 
- **Loss:** ~$500,000 
- **Post-mortem:** [Link to post-mortem](https://arxiv.org/abs/1812.05934)
   
## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107/)
- [OpenZeppelin ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard)
- [Slither Reentrancy Detection](https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities) 

---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Batch Reentrancy with Incomplete State 
severity: C
score:
impact: 5  
exploitability: 4 
reachability: 4  
complexity: 3   
detectability: 4  
finalScore: 4.35
```

---

## 📄 Justifications & Analysis

- **Impact**: Full contract drain or logic break if attacker loops reentry.
- **Exploitability**: High if attacker address is part of the batch.
- **Reachability**: Found in many DeFi, NFT, and vault payout patterns.
- **Complexity**: Intermediate – requires cross-function understanding.
- **Detectability**: Tools can detect it, but may be missed in subtle cases.