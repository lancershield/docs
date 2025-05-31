# Lack of Fallback Function

```YAML
id: LS12I
title: Lack of Fallback Function
baseSeverity: I
category: ether-receiving
language: solidity
blockchain: [ethereum]
impact: Rejected ETH transfers due to missing fallback/receive handler
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.6.0", "<0.8.23"]
cwe: CWE-703
swc: SWC-111
```

## 📝 Description

- A contract without a receive() or fallback() function cannot accept direct ETH transfers via send() or transfer() unless the transaction includes function data that matches an existing signature. 
- This omission can cause ETH sent to the contract to be rejected (and reverted), especially in integrations where third-party contracts or users attempt ETH transfers.
- In scenarios such as treasury contracts, fundraising contracts, or minimal proxies, the absence of a fallback mechanism can lead to operational failure, loss of funds (if sent by mistake), or user confusion.

## 🚨 Vulnerable Code

```solidity

// No fallback or receive function defined
contract Vault {
    mapping(address => uint) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint amount) public {
        require(balances[msg.sender] >= amount);
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit simulation:

1. A user or another contract calls address(vault).transfer(1 ether);.
2. Since the contract lacks a receive() or fallback() function, the transaction reverts.
3. If this transfer was intended as a payment, the operation fails and may cause disruption.
4. In protocols expecting passive ETH receipt (e.g., donations, royalties), funds are lost or stuck on the sender side.

**Assumptions:**

- ETH is sent directly via .transfer() or .send().
- No calldata or unmatched function selector is present.

## ✅ Fixed Code

```solidity

// Securely handle plain ETH transfers
contract Vault {
    mapping(address => uint) public balances;

    receive() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint amount) public {
        require(balances[msg.sender] >= amount);
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: I
  reasoning: "Affects ETH receiving under specific call conditions but not exploitable."
- context: "Donation or Vault Contract"
  severity: M
  reasoning: "Expected to passively receive ETH; failure results in operational loss."
- context: "Proxy Contract or Governance Contract"
  severity: H
  reasoning: "May block delegation, admin recovery, or proxy upgrades triggered by ETH fallback flows."
```

## 🛡️ Prevention

### Primary Defenses

- Always include a receive() function to handle plain ETH transfers.
- Optionally define a fallback() for future compatibility and graceful degradation.

### Additional Safeguards

- Use logging inside receive() to track unintended transfers.
- Return excess ETH with logic caps if necessary.

### Detection Methods

- Solidity compiler warnings when payable fallback is missing.
- Slither missing-receive detector.
- Solhint: no-empty-blocks, compiler-receive-payable.

## 🕰️ Historical Exploits

- **Name:** TokenVault Rejection Bug 
- **Date:** 2020-11-08 
- **Loss:** 14 ETH 
- **Post-mortem:** [Link to post-mortem](https://forum.openzeppelin.com/t/contract-cannot-receive-ether/2603) 
  
## 📚 Further Reading

- [SWC-111: Use of Deprecated Functions or Lack of Receive](https://swcregistry.io/docs/SWC-111/) 
- [Solidity Docs – Receive Ether Function](https://docs.soliditylang.org/en/latest/contracts.html#receive-ether-function) 
- [Slither: missing-receive Detector](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: LS12I
title: Lack of Fallback Function
severity: I
score:
impact: 2
exploitability: 1
reachability: 4
complexity: 1
detectability: 4
finalScore: 2.15
```

---

## 📄 Justifications & Analysis

- **Impact**: May cause failed payments or refunds, but no active loss or exploitability.
- **Exploitability**: Requires mistake or third-party transfer with incorrect assumptions.
- **Reachability**: ETH transfers from wallets or contracts are common.
- **Complexity**: Simple to understand and exploit (or trigger by accident).
- **Detectability**: Easily visible in code reviews or static analysis.
