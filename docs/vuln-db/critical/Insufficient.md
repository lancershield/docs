# Insufficient Signature Verification

```YAML
id: TBA
title: Insufficient Signature Verification
baseSeverity: C
category: authentication
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Unauthorized access, forged approvals, stolen funds
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-347
swc: SWC-117
```

## 📝 Description

- Insufficient signature verification vulnerabilities arise when a contract improperly checks digital signatures, either by:
- Failing to verify that the signer is the authorized party.
- Omitting replay protection (e.g., nonce).
- Allowing incorrect or incomplete message construction.
- This flaw can result in unauthorized access, fund transfers, or malicious relays of signed messages. 
- It commonly affects permit functions, off-chain authorization flows, and meta-transactions.

## 🚨 Vulnerable Code

```solidity
contract InsecureSigner {
    function execute(address target, bytes calldata data, bytes memory signature) external {
        bytes32 message = keccak256(abi.encodePacked(target, data));
        address signer = recoverSigner(message, signature);
        // ❌ No check to ensure signer is authorized
        (bool success, ) = target.call(data);
        require(success, "Execution failed");
    }

    function recoverSigner(bytes32 message, bytes memory sig) public pure returns (address) {
        return ECDSA.recover(ECDSA.toEthSignedMessageHash(message), sig);
    }
}
```
## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker generates a signature offline or reuses a leaked one.
2. Sends the signature with execute() call to the vulnerable contract.
3. Contract recovers a valid signer address but never checks if the signer is authorized.
4. Attacker hijacks execution to arbitrary target with crafted calldata.

**Assumptions:**

- Weak or incomplete signature verification can allow malicious actors to execute unauthorized transactions.
- If the contract does not enforce nonce or timestamp validation, replay attacks become feasible.

## ✅ Fixed Code

```solidity

contract SecureSigner {
    mapping(address => bool) public authorizedSigners;
    mapping(bytes32 => bool) public usedMessages;

    function execute(address target, bytes calldata data, bytes memory signature) external {
        bytes32 message = keccak256(abi.encodePacked(msg.sender, target, data));
        require(!usedMessages[message], "Replay detected");
        address signer = recoverSigner(message, signature);
        require(authorizedSigners[signer], "Invalid signer");
        usedMessages[message] = true;
        (bool success, ) = target.call(data);
        require(success, "Execution failed");
    }

    function recoverSigner(bytes32 message, bytes memory sig) public pure returns (address) {
        return ECDSA.recover(ECDSA.toEthSignedMessageHash(message), sig);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Permit-based approval, meta-transactions, or bridge authentication"
  severity: C
  reasoning: "Leads to total compromise of user assets or impersonation"
- context: "Testnet demo or UI-only logic"
  severity: L
  reasoning: "No meaningful risk beyond demonstration scope"
- context: "System uses EIP-712 with domain separation and nonce"
  severity: I
  reasoning: "Fully mitigated"
```

## 🛡️ Prevention

### Primary Defenses

- Always verify the identity of the recovered signer.
- Use nonces or message hashes to prevent replay attacks.
- Include contextual domain data (e.g., msg.sender, chainId, contract address) in the signed message.

### Additional Safeguards

- Implement EIP-712 structured data signatures for clarity and security.
- Use OpenZeppelin’s SignatureChecker and ECDSA libraries.
- Add expiry timestamps to signed messages.

### Detection Methods

- Manual review of recover and ecrecover logic.
- Static analysis using Slither (insecure-signature) or Mythril symbolic execution.
- Check for presence of replay protection and signer verification logic.

## 🕰️ Historical Exploits

- **Name:** Odos Protocol Exploit 
- **Date:** 2024-04-15 
- **Loss:** Approximately $50,000 
- **Post-mortem:** [Link to post-mortem](https://www.quillaudits.com/blog/hack-analysis/odos-protocol-arbitrary-call-vulnerability)  
- **Name:** Wormhole Bridge Hack 
- **Date:** 2022-02-02 
- **Loss:** Over $320 million 
- **Post-mortem:** [Link to post-mortem](https://www.nethermind.io/blog/smart-contract-vulnerabilities-and-mitigation-strategies) 

## 📚 Further Reading

- [SWC-122: Insecure Signature Verification](https://swcregistry.io/docs/SWC-122) 
- [OpenZeppelin – ECDSA Utilities](https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA)
- [EIP-712: Structured Data Hashing and Signing](https://eips.ethereum.org/EIPS/eip-712) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Insufficient Signature Verification 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 3    
detectability: 3  
finalScore: 4.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Can allow arbitrary contract execution, minting, or asset transfer.
- **Exploitability**: Easily triggered if signature recovery lacks validation.
- **Reachability**: Common in relayers, permit()-style flows, and governance voting.
- **Complexity**: Medium; requires signing knowledge, but scripts and tools exist.
- **Detectability**: Partially detectable via audit tools; full coverage needs manual review.
