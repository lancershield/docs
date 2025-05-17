# Unsafe tx.origin Usage 

```YAML
id: TBA
title: Unsafe tx.origin Usage 
severity: H
category: access-control
language: solidity
blockchain: [ethereum]
impact: Attacker can trick users into authorizing transactions that compromise their assets
status: draft
complexity: medium
attack_vector: phishing
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-640
swc: SWC-115
```

## 📝 Description

- Unsafe usage of `tx.origin` for authentication or permission checks allows attackers to exploit phishing-style proxy contracts that forward transactions from unsuspecting users. 
- Unlike `msg.sender`, which refers to the immediate caller, `tx.origin` refers to the original external account that initiated the transaction — even across multiple contract calls.
- When `tx.origin` is used to restrict access (e.g., `require(tx.origin == owner)`), a malicious contract can:
- Call the vulnerable contract via a proxy,
- Bypass `msg.sender` protections,
- And exploit user trust to drain funds or escalate privileges.

## 🚨 Vulnerable Code

```solidity
contract UnsafeAuth {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function withdraw() external {
        require(tx.origin == owner, "Not owner"); // ❌ vulnerable
        payable(msg.sender).transfer(address(this).balance);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. Attacker deploys a malicious contract with a fallback function that calls withdraw() on the vulnerable contract.
2. Attacker convinces the target owner to call the malicious contract (e.g., "claim your airdrop").
3. The malicious contract forwards the call to the target contract.
4. In the target contract, tx.origin == owner is true — withdraw succeeds, and funds are stolen.

**Assumptions:**

- Victim is the legitimate owner.
- They are tricked into initiating a transaction through a malicious intermediary.

## ✅ Fixed Code

```solidity

contract SafeAuth {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner"); // ✅ use msg.sender instead
        payable(msg.sender).transfer(address(this).balance);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Never use tx.origin for access control.
- Use msg.sender instead, which reflects the immediate caller.
- Use role-based access control libraries like OpenZeppelin’s Ownable or AccessControl.

### Additional Safeguards

- Educate users to avoid interacting with unverified contracts.
- Monitor for permission checks that depend on global transaction context.
- Avoid deeply nested contract calls where caller context may be ambiguous.

### Detection Methods

- Slither: tx-origin detector.
- Compiler warnings if tx.origin is used.
- Manual audit for any usage of tx.origin in require(...) or access checks.

## 🕰️ Historical Exploits

- **Name:** Wallet Contract Phishing Attack 
- **Date:** 2023 
- **Loss:** Complete wallet drain 
- **Post-mortem:** [Link to post-mortem](https://www.cyfrin.io/glossary/phishing-with-tx-origin-hack-solidity-code-example)  


## 📚 Further Reading

- [SWC-115: Authorization via tx.origin](https://swcregistry.io/docs/SWC-115) 
- [Solidity Docs – tx.origin](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#tx-origin) 
- [Slither – tx.origin Detector](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Unsafe tx.origin Usage 
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 3   
complexity: 2     
detectability: 5  
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: High — attacker can gain unauthorized control over withdrawals or config changes.
- **Exploitability**: Feasible via phishing or malicious dApp interaction.
- **Reachability**: Limited to contracts that explicitly misuse tx.origin.
- **Complexity**: Moderate — attack setup is simple, but user interaction is required.
- **Detectability**: Very high — static tools like Slither catch this reliably.