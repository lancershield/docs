# Ignoring ChainID in Signatures

```YAML
id: TBA
title: Ignoring ChainID in Signatures 
severity: H
category: signature-verification
language: solidity
blockchain: [ethereum]
impact: Valid signatures from one chain may be replayed on another chain
status: draft
complexity: medium
attack_vector: cross-chain
mitigation_difficulty: easy
versions: [">=0.6.0", "<latest"]
cwe: CWE-347
swc: SWC-121
```

## 📝 Description

- Ignoring `chainId` in signature validation allows malicious actors to replay valid signed messages across different EVM-compatible chains. 
- When signature schemes do not bind the signed message to a specific chain, the same signed payload (e.g., `approve()`, `mint()`, `withdraw()`, `executeMetaTx()`) can be reused on another chain, resulting in:
- Duplicate executions,
- Drained balances or unauthorized operations,
- Critical multi-chain desynchronization bugs.

## 🚨 Vulnerable Code

```solidity
function executeMetaTx(address user, bytes calldata data, bytes calldata signature) external {
    bytes32 hash = keccak256(abi.encodePacked(user, data)); // ❌ No chainId binding
    require(recover(hash, signature) == user, "Invalid signature");

    (bool success, ) = address(this).call(data);
    require(success, "Execution failed");
}
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. A user signs a valid meta-transaction on Chain A (e.g., Polygon).
2. Attacker captures the signature and resubmits it on Chain B (e.g., BNB Chain).
3. Since the message is valid and chainId was not part of the hash, it executes successfully.
4. If the protocol uses the same contract on both chains, attacker replays the action (e.g., withdraw, claim, mint) and drains tokens on the second chain.

**Assumptions:**

- Signature payload is not bound to a specific chainId.
- The same contract logic exists on multiple chains with similar state.

## ✅ Fixed Code

```solidity

function getMetaTxHash(address user, bytes calldata data, uint256 nonce, uint256 chainId) public pure returns (bytes32) {
    return keccak256(abi.encodePacked(user, data, nonce, chainId));
}

function executeMetaTx(address user, bytes calldata data, uint256 nonce, bytes calldata signature) external {
    bytes32 hash = getMetaTxHash(user, data, nonce, block.chainid); // ✅ Includes chainId
    require(recover(hash, signature) == user, "Invalid signature");

    // Execute logic
}
```

## 🛡️ Prevention

### Primary Defenses

- Always include block.chainid (or EIP-712 domain with chain ID) in the signed message.
- Use EIP-712 domain separators that bind to:
- name
- version
- chainId
- verifyingContract

### Additional Safeguards

- Reject messages where chainId does not match block.chainid.
- Monitor for repeated signatures across chains (using bridges or relayer infra).
- Use EIP-712 or eth_signTypedDataV4 for all critical signature flows.

### Detection Methods

- Slither: missing-chainid-in-hash, signature-replay-cross-chain detectors.
- Manual audit of all signature logic for keccak256(...) and ecrecover(...).
- Test signature replay scenarios across forked chain environments.

## 🕰️ Historical Exploits

- **Name:** EPProgramManager Replay Vulnerability 
- **Date:** November 2024 
- **Loss:** Not publicly disclosed 
- **Post-mortem:** [Link to post-mortem](https://github.com/sherlock-audit/2024-11-superfluid-locking-contract-judging/issues/6) 

## 📚 Further Reading

- [SWC-121: Missing Protection Against Signature Replay](https://swcregistry.io/docs/SWC-121) 
- [EIP-712: Typed Structured Data Hashing](https://eips.ethereum.org/EIPS/eip-712) 
- [EIP-2612: Permit with Nonce and ChainID](https://eips.ethereum.org/EIPS/eip-2612) 
- [Slither – Signature Replay & Domain Detectors](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Ignoring ChainID in Signatures 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 3   
complexity: 2     
detectability: 4  
finalScore: 4.2
```

---

## 📄 Justifications & Analysis

- **Impact**: Critical — cross-chain replay can lead to theft or duplicated actions.
- **Exploitability**: Trivial for an attacker once the signature is observed.
- **Reachability**: Found in many multi-chain deployments and unguarded signature flows.
- **Complexity**: Simple to fix but easy to miss during development.
- **Detectability**: High — static tools and manual review will surface it.