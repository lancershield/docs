# Domain Separator Collision 

```YAML
id: TBA
title: Domain Separator Collision 
baseSeverity: H
category: cryptography
language: solidity
blockchain: [ethereum]
impact: Signature reuse, unauthorized execution, or replay attacks
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-294
swc: SWC-136
```

## 📝 Description

- EIP-712 defines typed structured data hashing for signing messages in Ethereum. 
- A key component of its design is the domain separator, which isolates signed data across different contracts or applications. If multiple contracts share the same domain separator, then signatures valid in one context can be replayed in another, breaking signature uniqueness and enabling replay attacks.
- A domain separator collision occurs when:
- The domainSeparator() is not uniquely derived using all expected parameters (e.g., name, version, chainId, and contract address)
- The domain values are hardcoded or improperly reused across contracts or chains
- This flaw can allow an attacker to reuse a signature across multiple contracts or chains to execute unauthorized actions.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract InsecurePermit {
    bytes32 public DOMAIN_SEPARATOR = keccak256(abi.encode(
        keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
        keccak256(bytes("PermitContract")),
        keccak256(bytes("1")),
        1,                   // ❌ Hardcoded chainId
        address(this)
    ));

    // permit() uses DOMAIN_SEPARATOR...
}
```

## 🧪 Exploit Scenario

1. Alice signs a message approving a spender in PermitContractA on Ethereum Mainnet.
2. PermitContractB on another chain or fork uses the exact same DOMAIN_SEPARATOR.
3. Attacker uses the signed permit in PermitContractB to steal tokens or gain authorization.
4. Alice is unaware her signature was reused across environments.

**Assumptions:**

- Contracts do not include chainId dynamically via block.chainid.
- The same contract bytecode is deployed with the same name/version on multiple chains or forks.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SecurePermit {
    bytes32 public immutable DOMAIN_SEPARATOR;

    constructor() {
        DOMAIN_SEPARATOR = keccak256(abi.encode(
            keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
            keccak256(bytes("PermitContract")),
            keccak256(bytes("1")),
            block.chainid,         // ✅ Use dynamic chainId
            address(this)
        ));
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Cross-chain signature validation for token transfers or approvals"
  severity: H
  reasoning: "May lead to replayed approvals or asset loss across chains"
- context: "Local-only domain with no cross-chain replay risk"
  severity: M
  reasoning: "Impact limited to fork scenarios or redeploys"
- context: "Uses EIP712.sol or dynamic domain separator with full context"
  severity: I
  reasoning: "Proper implementation; no practical exploit path"
```

## 🛡️ Prevention

### Primary Defenses

- Always include block.chainid and address(this) dynamically in the domain separator.
- Never hardcode domain values unless verified to be unique.

### Additional Safeguards

- Append unique identifiers to the name or version field when reusing codebases.
- Maintain a registry of deployed domain separators if managing multiple contracts.

### Detection Methods

- Audit DOMAIN_SEPARATOR calculation logic.
- Identify reused hardcoded domain components.
- Tools: Slither (eip712-domain), MythX, custom linters

## 🕰️ Historical Exploits

- **Name:** SushiSwap xDomain Replay Risk (Layer 2) 
- **Date:** 2021 
- **Loss:** None (caught pre-deployment) 
- **Post-mortem:** [OpenZeppelin xDomain Risk Warning](https://forum.openzeppelin.com/t/metatransactions-and-eip-712-replay-attacks-across-chains/5863)

  
## 📚 Further Reading

- [SWC-136: Unencrypted Sensitive Data](https://swcregistry.io/docs/SWC-136/) 
- [EIP-712: Typed Structured Data Hashing](https://eips.ethereum.org/EIPS/eip-712)
- [OpenZeppelin – EIP-712 Domain Separator Best Practices](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712) 
- [MetaMask – Signature Replay Risks](https://docs.metamask.io/guide/signing-data.html#signtypeddata) 

---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Domain Separator Collision 
severity: H
score:
impact: 4        
exploitability: 4 
reachability: 4  
complexity: 2    
detectability: 3  
finalScore: 3.7
```

---

## 📄 Justifications & Analysis

- **Impact**: Replay across contracts or chains undermines the entire trust model of EIP-712.
- **Exploitability**: Easy if attacker controls or observes valid signatures in reused environments.
- **Reachability**: Common in permit() functions, signatures, and meta-transactions.
- **Complexity**: Easy to implement incorrectly, especially with hardcoded constants.
- **Detectability**: Subtle unless specifically reviewing domain separator components.
