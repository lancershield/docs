# Permit Replay Across Chains

```YAML
id: TBA
title: Permit Replay Across Chains 
baseSeverity: H
category: signature-authentication
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc, avalanche]
impact: Unauthorized token approvals or fund movements on alternate chains
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-347
swc: SWC-122
```

## 📝 Description

- The permit() function in ERC-2612 enables gasless token approvals via off-chain signatures. However, when a project deploys the same token contract (same address and domain separator) across multiple chains, an attacker can replay a signature signed for one chain on another, because:
- The signature does not include the chain ID or domain separator tied to a unique network.
- EIP-712 DOMAIN_SEPARATOR values are identical across chains if contract addresses and names are reused.
- This allows an attacker to replay an approval on a different network, enabling unauthorized token usage or approval.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract ERC20WithPermit {
    mapping(address => uint256) public nonces;

    function permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v, bytes32 r, bytes32 s
    ) external {
        require(block.timestamp <= deadline, "Expired");

        bytes32 digest = keccak256(
            abi.encodePacked(
                "\x19\x01",
                DOMAIN_SEPARATOR(), // ❌ same across chains if not unique
                keccak256(abi.encode(
                    keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
                    owner, spender, value, nonces[owner]++, deadline
                ))
            )
        );

        address signer = ecrecover(digest, v, r, s);
        require(signer == owner, "Invalid signature");

        _approve(owner, spender, value);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A user signs a valid permit on Chain A authorizing a spender to spend 100 tokens.
2. The attacker sees the permit signature and recognizes that the token contract is also deployed on Chain B at the same address and with the same domain separator config.
3. The attacker submits the same permit() call on Chain B using the original signature.
4. Since the contract’s DOMAIN_SEPARATOR() is identical and nonce tracking is separate per chain, the permit is accepted again.
5. The attacker gains approval to spend 100 tokens on Chain B, despite the user never signing a message for that network.

**Assumptions:**

- The same private key signs permits across chains.
- Tokens are deployed with same name/symbol and contract address.
- DOMAIN_SEPARATOR() lacks chain ID discrimination.

## ✅ Fixed Code

```solidity

function DOMAIN_SEPARATOR() public view returns (bytes32) {
    return keccak256(abi.encode(
        keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
        keccak256(bytes("MyToken")),
        keccak256(bytes("1")),
        block.chainid, // ✅ unique per chain
        address(this)
    ));
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Tokens can be drained or misused across multiple chains if replayable permits are accepted."
- context: "Tokens deployed on single chain only"
  severity: M
  reasoning: "Risk is limited to a single environment, reducing likelihood of exploit."
- context: "Cross-chain bridges and multi-chain deployments"
  severity: C
  reasoning: "Highly critical in ecosystems where same contracts exist across chains. Replay results in 
```

## 🛡️ Prevention

### Primary Defenses

- Always include block.chainid in EIP712Domain construction.
- Avoid reusing the same token address and name across chains without accounting for domain uniqueness.

### Additional Safeguards

- Log the EIP712 domainSeparator() in tests and compare across chains.
- Use verifyDomainSeparator() as a fuzzing invariant during tests.

### Detection Methods

- Static check for DOMAIN_SEPARATOR lacking block.chainid
- Manual validation across chain deployments
- Tools: Slither (eip712-misuse), custom EIP712 domain diff scripts

## 🕰️ Historical Exploits

- **Name:** Multichain Cross-Chain Replay Attack 
- **Date:** 2023-07 
- **Loss:** ~$28,520 
- **Post-mortem:** [Link to post-mortem](https://arxiv.org/html/2504.07589v1) 
- **Name:** Moonwell Cross-Chain Permit Vulnerability 
- **Date:** 2023-07 
- **Loss:** Not publicly disclosed 
- **Post-mortem:** [Link to post-mortem](https://www.halborn.com/audits/moonwell/contracts-v2-updates)
  
## 📚 Further Reading

- [SWC-122: Signature Malleability](https://swcregistry.io/docs/SWC-122) 
- [CWE-347: Improper Verification of Cryptographic Signature](https://cwe.mitre.org/data/definitions/347.html) 
- [EIP-2612 Permit Standard](https://eips.ethereum.org/EIPS/eip-2612)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Permit Replay Across Chains 
severity: H
score:
impact: 5        
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 3  
finalScore: 4.2
```

---

## 📄 Justifications & Analysis

- **Impact**: Enables silent cross-chain token authorization
- **Exploitability**: Readily abused by capturing a single signed message
- **Reachability**: Present in many multichain deployments with reused codebases
- **Complexity**: Relatively subtle; requires understanding of EIP-712 internals
- **Detectability**: Not obvious unless chain IDs and domain hashes are explicitly compared