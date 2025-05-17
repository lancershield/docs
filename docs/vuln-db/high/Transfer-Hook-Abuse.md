# Transfer Hook Abuse

```YAML
id: TBA
title: Transfer Hook Abuse 
severity: H
category: token-logic
language: solidity
blockchain: [ethereum]
impact: Unexpected logic may be triggered during transfer, mint, or burn via malicious receiver
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-610
swc: SWC-135
```

## 📝 Description

- Transfer hook abuse occurs when smart contracts implementing ERC20, ERC721, or ERC1155 standards call arbitrary logic inside `_beforeTokenTransfer`, `_afterTokenTransfer`, or `onERC721Received`, which may be controlled by malicious contracts or influenced by attackers. These hooks can:
- Trigger reentrancy during transfers or minting,
- Allow unexpected state changes mid-execution,
- Be exploited to block transfers via gas griefing or revert loops,
- Create infinite mint/loop effects if the hook itself calls `mint()` or `transfer()` recursively.
- This is especially dangerous in tokens with rich on-transfer logic (e.g., reflection tokens, taxes, vaults, LP tokens).

## 🚨 Vulnerable Code

```solidity
contract HookedToken is ERC20 {
    function _afterTokenTransfer(address from, address to, uint256 amount) internal override {
        // ❌ Arbitrary logic triggered after every transfer
        IReward(to).onReceive(from, amount); 
    }
}
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. Attacker deploys a contract with a onReceive() function that re-enters transfer() or calls back into the token.
2. Attacker receives a transfer from the token.
3. onReceive() is called via _afterTokenTransfer hook, which triggers further logic.
4. The token is re-entered in an unsafe state or logic is bypassed, allowing:

**Assumptions:**

- onReceive() is not protected by reentrancy guards or gas limits.
- Token calls external, untrusted logic during transfer hooks.

## ✅ Fixed Code

```solidity

contract SafeHookedToken is ERC20, ReentrancyGuard {
    function _afterTokenTransfer(address from, address to, uint256 amount) internal override nonReentrant {
        if (isTrusted(to)) {
            IReward(to).onReceive(from, amount); // ✅ Restricted and guarded
        }
    }

    function isTrusted(address to) internal view returns (bool) {
        return whitelistedReceivers[to];
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Do not call arbitrary or untrusted external contracts inside transfer hooks.
- Use nonReentrant modifiers if needed in hook logic.
- Whitelist addresses that are allowed to receive hook calls.

### Additional Safeguards

- Use try/catch to handle external calls inside hooks defensively.
- Add gas limits if using low-level calls (e.g., call{gas: 50000}).
- Document all transfer hook behavior clearly in contract interface.

### Detection Methods

- Slither: external-calls-in-transfer-hooks, transfer-hook-reentrancy, unsafe-transfer-hook detectors.
- Manual audits of _beforeTokenTransfer, _afterTokenTransfer, onERC721Received, onERC1155Received.
- Fuzz tests that simulate malicious receiver contracts.

## 🕰️ Historical Exploits

- **Name:** C.R.E.A.M. Finance AMP Token Exploit 
- **Date:** August 31, 2021 
- **Loss:** Approximately $18.8 million (462 million AMP tokens and 2,804.96 ETH) 
- **Post-mortem:** [Link to post-mortem](https://medium.com/cream-finance/c-r-e-a-m-finance-post-mortem-amp-exploit-6ceb20a630c5) 
  

## 📚 Further Reading

- [SWC-135: Unexpected Behavior](https://swcregistry.io/docs/SWC-135) 
- [Solidity Docs – Transfer Hooks](https://docs.soliditylang.org/en/latest/units-and-global-variables.html) 
- [OpenZeppelin Transfer Hook Guidelines](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_afterTokenTransfer-address-address-uint256-) 
- [Slither Transfer Hook Detectors](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Transfer Hook Abuse 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2    
detectability: 5  
finalScore: 4.4
```

---

## 📄 Justifications & Analysis

- **Impact**: Critical in vaults, reward systems, and tax tokens — can break invariants or enable theft.
- **Exploitability**: Any user can deploy a malicious contract to exploit the hook unless restricted.
- **Reachability**: Common pattern in modern DeFi tokens and yield contracts.
- **Complexity**: Misuse is simple, but consequences are complex.
- **Detectability**: Readily detected via audits and Slither-based static analysis.