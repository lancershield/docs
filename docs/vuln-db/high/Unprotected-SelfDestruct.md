# Unprotected SelfDestruct

```YAML
id: TBA
title: Unprotected SelfDestruct
severity: H
category: lifecycle
language: solidity
blockchain: [ethereum, bsc, polygon, arbitrum, optimism]
impact: Permanent deletion of contract logic and storage
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-284: Improper Access Control
swc: SWC-106: Unprotected Selfdestruct Instruction
```

## 📝 Description

- The selfdestruct() opcode allows a contract to delete its code and storage from the blockchain and optionally forward remaining ETH to an address. When this function is exposed without proper access control or deactivation after deployment, it can be exploited to permanently kill a contract.
- Consequences include:
- Destruction of critical business logic (rendering contracts unusable)
- Loss of user funds (if balances are held in the contract)
- Malicious denial-of-service (DoS) for apps relying on the contract
- Even if selfdestruct() is guarded by onlyOwner, risks persist if the owner’s key is compromised or if governance-controlled execution paths allow invoking destructors.

## 🚨 Vulnerable Code

```solidity

function destroy() external {
    selfdestruct(payable(msg.sender)); 
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A DeFi staking contract has a destroy() function meant for migration purposes.
2. The developer forgets to add an access control modifier (e.g., onlyOwner).
3. An attacker calls destroy(), triggering selfdestruct().
4. The contract is removed from the blockchain
5. Attackers monitor deployed bytecode or ABIs for exposed selfdestruct logic.

## ✅ Fixed Code

```solidity

function destroy() external onlyOwner {
    selfdestruct(payable(owner)); // ✅ access-controlled
}
```

## 🛡️ Prevention

### Primary Defenses

- Never include selfdestruct() in production contracts unless absolutely necessary.
- If included, strictly guard with onlyOwner, onlyTimelock, or DAO governance.
- Use upgradeable proxies if upgrade/migration is needed.

### Additional Safeguards

- Remove migration/test-only destructors before deployment.
- Review contract bytecode and ABI for presence of selfdestruct.

### Detection Methods

- Slither: dangerous-low-level, unprotected-selfdestruct
- Custom script: scan bytecode for 0xff (opcode for selfdestruct)
- Foundry fuzzing: simulate destroy() exposure

## 🕰️ Historical Exploits

- **Name:** Parity Multisig Wallet Library SelfDestruct 
- **Date:** 2017-11 
- **Loss:** ~$150M 
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert.html) 

## 📚 Further Reading

- [SWC-106: Unprotected Selfdestruct Instruction](https://swcregistry.io/docs/SWC-106) 
- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html) 
- [Solidity Docs – selfdestruct](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#deactivate-and-selfdestruct)  

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Unprotected SelfDestruct
severity: H
score:
impact: 5 
exploitability: 4  
reachability: 4  
complexity: 1  
detectability: 5  
finalScore: 4.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Results in total contract loss and user fund inaccessibility; affects downstream dApps and integrations.
- **Exploitability**: Requires only one function call if unguarded; even owner-guarded versions pose risk if compromised.
- **Reachability**: Externally accessible in many contracts unless protected; also found in older templates.
- **Complexity**: Low technical skill needed to exploit.
- **Detectability**: Easily caught in audits or static tools like Slither or hardhat plugins.

