# Signature Replay Attacks

```YAML
id: TBA
title: Signature Replay Attacks due to Missing Nonce or Domain Separation
severity: H
category: signature-replay
language: solidity
blockchain: [ethereum]
impact: Unauthorized reuse of signed messages for repeated exploitation
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-294
swc: SWC-121
```


## 📝 Description

- Signature replay attacks occur when a valid off-chain signature can be reused multiple times to trigger the same or similar actions across **time, chains, or contracts**. - This typically happens due to:
- Lack of **nonces**,
- Absence of **expiry timestamps**,
- No use of **domain separation** (e.g., `chainId`, `contract address`).
- The attack allows unauthorized repetition of actions such as token approvals, withdrawals, or function executions with previously valid signatures.

## 🚨 Vulnerable Code

```solidity
contract ReplayVuln {
    mapping(address => bool) public claimed;

    function claimReward(bytes calldata signature) external {
        bytes32 message = keccak256(abi.encodePacked(msg.sender));
        address signer = recoverSigner(message, signature);
        require(signer == address(0xAbC...123), "Invalid signer");

        // ❌ No nonce or timestamp → signature can be reused
        claimed[msg.sender] = true;
        // Reward logic...
    }

    function recoverSigner(bytes32 message, bytes memory sig) internal pure returns (address) {
        return ECDSA.recover(ECDSA.toEthSignedMessageHash(message), sig);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A user receives a signed message authorizing them to claim a reward.
2. Attacker intercepts or duplicates the signature.
3. Calls claimReward() using the same signature multiple times (or on multiple chains).
4. Contract has no nonce or expiry check—replay succeeds repeatedly.

**Assumptions:**

- No used flag, nonce, or expiry is validated.
- Signature verification checks only the signer, not the context.

## ✅ Fixed Code

```solidity

contract ReplaySafe {
    mapping(bytes32 => bool) public usedMessages;

    function claimReward(bytes calldata signature, uint256 nonce) external {
        bytes32 message = keccak256(abi.encodePacked(msg.sender, nonce, address(this), block.chainid));
        require(!usedMessages[message], "Replay detected");

        address signer = recoverSigner(message, signature);
        require(signer == 0xAbC...123, "Invalid signer");

        usedMessages[message] = true;
        // Reward logic...
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use nonces or message hashes and mark them as used after execution.
- Include domain separation (e.g., contract address, chainId) in the signed message.
- Add expiry timestamps to signed payloads to limit the attack window.

### Additional Safeguards

- Use EIP-712 structured data signing to enforce well-defined domain scopes.
- Reject duplicate messages at the storage level (usedMessages mapping).
- Ensure off-chain signer follows correct hashing schema.

### Detection Methods

- Manual inspection of recover() logic for absence of nonce/domain/expiry.
- Slither: insecure-signature, missing-nonce patterns.
- Symbolic fuzzing on claim, permit, or meta-transaction flows.

## 🕰️ Historical Exploits

- **Name:** Bee Token ICO Replay Phishing 
- **Date:** 2018-02 
- **Loss:** ~$1M in replayed/phished contributions 
- **Post-mortem:** [Link to post-mortem](https://thehackernews.com/2018/02/bee-token-phishing-scam.html) 



## 📚 Further Reading

- [SWC-121: Signature Replay](https://swcregistry.io/docs/SWC-121) 
- [EIP-712: Typed Structured Data Signing](https://eips.ethereum.org/EIPS/eip-712) 
- [OpenZeppelin – Preventing Signature Replay](https://docs.openzeppelin.com/contracts/4.x/utilities#ECDSA) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Signature Replay Attacks 
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.0
```


---

## 📄 Justifications & Analysis

- **Impact**: Exploiter can claim rewards, approve transfers, or act on behalf of user multiple times.
- **Exploitability**: Requires only access to a signed message; no complex tooling.
- **Reachability**: Affects any off-chain signed authorization-based flow (claim(), permit(), etc.).
- **Complexity**: Medium — replay just involves resubmitting a previous payload.
- **Detectability**: Signature flows are high-risk and easily audited for nonce/domain logic.
