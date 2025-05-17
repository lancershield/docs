# Missing Nonce Checks in Meta Transactions

```YAML
id: TBA
title: Missing Nonce Checks in Meta Transactions 
severity: H
category: signature-verification
language: solidity
blockchain: [ethereum]
impact: Reusable signatures allow duplicate execution of user-authorized actions
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-294
swc: SWC-122
```

## 📝 Description

- Missing nonce checks in meta transactions allows signatures to be replayed multiple times by anyone who sees them. 
- Without a unique, single-use nonce value tied to the signature, an attacker can:
- Replay signed messages across sessions, chains, or relayers,
- Drain user tokens, execute unauthorized contract calls,
- Or trigger unintended operations multiple times.
- This vulnerability commonly affects contracts using:
- `permit()` or `executeMetaTx()`-style flows,
- Custom signature-based authorization without EIP-2612-style nonces.

## 🚨 Vulnerable Code

```solidity
contract MetaTxBad {
    function executeMetaTx(
        address user,
        address to,
        bytes calldata data,
        bytes memory signature
    ) external {
        bytes32 hash = keccak256(abi.encodePacked(user, to, data));
        require(recover(hash, signature) == user, "Invalid signature");

        (bool success, ) = to.call(data); // ❌ No nonce used — replayable
        require(success, "Call failed");
    }
}
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. A user signs a valid meta transaction for transferring funds.
2. Attacker relays this message once — action succeeds.
3. Attacker then resubmits the same signed message multiple times.
4. Without a nonce check, all executions succeed, draining funds or triggering duplicate logic.

**Assumptions:**

- Signature does not bind to a user-specific, incrementing nonce.
- No expiration or unique salt mechanism is used.

## ✅ Fixed Code

```solidity

contract MetaTxGood {
    mapping(address => uint256) public nonces;

    function executeMetaTx(
        address user,
        address to,
        bytes calldata data,
        uint256 nonce,
        bytes memory signature
    ) external {
        require(nonce == nonces[user], "Invalid nonce");

        bytes32 hash = keccak256(abi.encodePacked(user, to, data, nonce));
        require(recover(hash, signature) == user, "Invalid signature");

        nonces[user] += 1;

        (bool success, ) = to.call(data);
        require(success, "Call failed");
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use nonces per user to prevent signature reuse.
- Increment nonce only after successful execution.
- Consider adding deadline or chainId to the signed message.

### Additional Safeguards

- Adopt EIP-712 typed data encoding to prevent cross-domain replays.
- Use OpenZeppelin’s ERC20Permit or ERC2771Context for secure patterns.
- Prevent nonce resets on upgrades or proxy reinitialization.

### Detection Methods

- Slither: signature-replay, missing-nonce-check, permit-replayable detectors.
- Manual review of keccak256(...), ecrecover(...) usage without unique identifier binding.
- Test cases using signature replay simulation.

## 🕰️ Historical Exploits

- **Name:** Ondo Finance KYC Signature Replay Vulnerability 
- **Date:** 2023 
- **Loss:** Potential for unauthorized KYC approvals 
- **Post-mortem:** [Link to post-mortem](https://dacian.me/signature-replay-attacks) 

## 📚 Further Reading

- [SWC-122: Signature Replay](https://swcregistry.io/docs/SWC-122) 
- [EIP-2612: permit() with nonce](https://eips.ethereum.org/EIPS/eip-2612) 
- [EIP-712: Typed Structured Data Signing](https://eips.ethereum.org/EIPS/eip-712) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Missing Nonce Checks in Meta Transactions 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 3    
detectability: 4  
finalScore: 4.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Replay can allow duplicate withdrawals, unauthorized actions, or draining of approvals.
- **Exploitability**: Straightforward if nonce is not checked and signature leaks.
- **Reachability**: Common in dApps using custom meta transaction relayers or gasless flows.
- **Complexity**: Moderate — requires signature crafting but can be automated.
- **Detectability**: High — static analyzers and audit checklists can flag missing nonce binding.