# Contract Address Collision

```YAML
id: TBA
title: Contract Address Collision 
baseSeverity: C
category: deployment
language: solidity
blockchain: [ethereum]
impact: Malicious contract may be deployed at predictable address to hijack trust assumptions
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-706
swc: SWC-127
```

## 📝 Description

- Contract address collision refers to the predictable deployment of a contract to a known address, allowing an attacker to pre-deploy a malicious contract at that location before or after the legitimate contract is deployed. This occurs when:
- A contract address is derived from a known deployer and nonce (`CREATE`),
- From a salt and deployer (`CREATE2`),
and the protocol logic or frontend assumes trust based on that address.
- Implications include:
- Hijacking initialization logic (`initialize()`),
- Pretending to be a trusted proxy or logic contract,
- Causing the system to break due to address-based trust assumptions.

## 🚨 Vulnerable Code

```solidity
// Assumes trusted logic at precomputed address
address logic = computeCreate2Address(salt, bytecodeHash);
Proxy(proxy).upgradeTo(logic); // ❌ No verification of logic contract owner
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. Protocol computes the future address of a logic contract using CREATE2 or CREATE.
2. Attacker front-runs and deploys their malicious contract to that address first.
3. When the protocol deploys the proxy and sets the implementation, it uses the malicious contract.
4. Attacker gains control over proxy logic or triggers arbitrary behavior via fallback/upgrade logic.

### Assumptions:

- Address is predictable and not protected.
- Protocol trusts any contract at that address without verifying ownership, code hash, or event signature.

## ✅ Fixed Code

```solidity

// Always verify code hash or bytecode before trusting precomputed addresses
address logic = computeCreate2Address(salt, bytecodeHash);
require(logic.code.length > 0, "Implementation contract not deployed");
require(keccak256(logic.code) == expectedHash, "Untrusted implementation");

// Prefer deploying logic contract first, then passing actual address
Proxy(proxy).upgradeTo(address(logic));
```

## 🧭 Contextual Severity

```yaml
- context: "Upgradable proxy system with deterministic admin or logic addresses"
  severity: C
  reasoning: "An attacker can hijack proxy paths or governance entry points."
- context: "User-specific vaults or wallets via predictable CREATE2"
  severity: H
  reasoning: "Attacker can impersonate contracts and steal funds."
- context: "Testing or isolated networks with low deployment risk"
  severity: L
  reasoning: "Impact is minimal in controlled environments."
```

## 🛡️ Prevention

### Primary Defenses

- Never assume a contract at a given address is safe without verifying:
- Code hash
- Ownership
- Event logs (deployment logs)
- Use CREATE2 only after deploying dependent logic first.

### Additional Safeguards

- Track deployed contracts in a registry or event log, not hardcoded addresses.
- Validate that contracts have expected storage layout and interfaces before assigning trust.
- Delay critical initialization if code hash mismatch is detected.

### Detection Methods

- Slither: predictable-deployment, upgrade-unverified, unsafe-create2-address detectors.
- Manual audit of logic involving create2, upgradeTo(...), and precomputed addresses.
- Deploy test contracts with same salt and verify deterministic address matches.

## 🕰️ Historical Exploits

- **Name:** Storage Collision Vulnerability in Ethereum Smart Contracts 
- **Date:** 2024 
- **Loss:** $12 million 
- **Post-mortem:** [Link to post-mortem](https://www.ndss-symposium.org/ndss-paper/not-your-type-detecting-storage-collision-vulnerabilities-in-ethereum-smart-contracts/) 


## 📚 Further Reading

- [SWC-127: Signature Replay / Address Collision](https://swcregistry.io/docs/SWC-127) 
- [Solidity Docs – create2](https://docs.soliditylang.org/en/latest/control-structures.html#salted-contract-creations-create2) 
- [OpenZeppelin Upgrade Patterns](https://docs.openzeppelin.com/upgrades-plugins) 
- [Ethereum Yellow Paper – CREATE2 Opcode](https://ethereum.org/en/developers/docs/evm/opcodes/#create2) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Contract Address Collision 
severity: C
score:
impact: 5         
exploitability: 4 
reachability: 3   
complexity: 3     
detectability: 5  
finalScore: 4.2
```

---

## 📄 Justifications & Analysis

- **Impact**: High — attacker can take over logic, upgrades, or protocol flow entirely.
- **Exploitability**: High if attacker can deploy to predictable address before the protocol.
- **Reachability**: Common in modern factory deployments and proxy systems.
- **Complexity**: Moderate — depends on knowledge of nonce/salt but feasible.
- **Detectability**: High — tooling can precompute all vulnerable addresses.