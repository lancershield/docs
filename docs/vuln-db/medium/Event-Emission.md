# Event Emission Flaws

```YAML
id: TBA
title: Event Emission Flaws Leading to Mismatched or Missing On-Chain Logs
severity: M
category: event-logging
language: solidity
blockchain: [ethereum]
impact: Off-chain systems misinterpret state changes or user actions
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-749
swc: SWC-136
```

## 📝 Description

- Event Emission Flaws occur when a smart contract fails to emit important events, emits them with incorrect data, or emits them in the wrong order. 
- While they do not affect on-chain state directly, these flaws can cause off-chain systems (indexers, dApps, or UIs) to display incorrect information, miss important actions (like token transfers), or falsely assume user behavior.
- This can affect compliance, auditing, dApp UX, and security monitors, especially when events are relied upon for reconstructing state.

## 🚨 Vulnerable Code

```solidity
contract Token {
    mapping(address => uint256) public balances;

    event Transfer(address indexed from, address indexed to, uint256 value);

    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        balances[to] += amount;

        // ❌ Incorrectly omits `from` field in event
        emit Transfer(address(0), to, amount);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. User A transfers tokens to user B.
2. Contract emits Transfer(address(0), B, amount) instead of Transfer(A, B, amount).
3. Off-chain systems interpret this as a mint instead of a transfer, corrupting analytics or triggering false alerts.
4. In other cases, if no event is emitted, off-chain balances may appear stale or incorrect.

**Assumptions:**

- Off-chain infrastructure (e.g., The Graph, Dune, Wallet UIs) depends on event logs.
- Emitted events contain inaccurate or missing fields.

## ✅ Fixed Code

```solidity

function transfer(address to, uint256 amount) external {
    require(balances[msg.sender] >= amount, "Insufficient balance");
    balances[msg.sender] -= amount;
    balances[to] += amount;

    emit Transfer(msg.sender, to, amount); // ✅ Correct emission
}
```

## 🛡️ Prevention

### Primary Defenses

- Always emit events after successful state changes (post-conditions).
- Include all required fields, especially indexed ones, in proper order.
- Follow ERC standards (e.g., ERC20 Transfer, Approval) exactly if applicable.

### Additional Safeguards

- Use unit tests or integration tests that track emitted events.
- Have off-chain indexers monitor for unexpected/missing event behavior.
- Lint or statically analyze contracts for unused or misordered event emissions.

### Detection Methods

- Slither: missing-events, incorrect-logging, event-parameter-mismatch detectors.
- Hardhat + Waffle tests for expectEvent() coverage.
- Review diff of emitted logs against expected functional behavior.

## 🕰️ Historical Exploits

- **Name:** MakerDAO Oracles Event Emission Delay 
- **Date:** 2020 
- **Loss:** Unspecified; potential data inconsistencies 
- **Post-mortem:** [Link to post-mortem](https://blog.makerdao.com/makerdao-oracle-delay-incident-report/)  

## 📚 Further Reading

- [SWC-136: Unexpected Ether Balance Change or Misleading Events](https://swcregistry.io/docs/SWC-136) 
- [Solidity Docs – Events](https://docs.soliditylang.org/en/latest/contracts.html#events) 
- [OpenZeppelin Docs – ERC20 Events](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256) 
  
---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Event Emission Flaws 
severity: M
score:
impact: 3         
exploitability: 2 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Doesn't affect on-chain state directly but can cause serious misrepresentation off-chain.
- **Exploitability**: Often due to human error or misunderstanding of event format.
- **Reachability**: Found in nearly every user interaction function (e.g., token transfers).
- **Complexity**: Very easy mistake; can occur in new token implementations or clones.
- **Detectability**: Audits and test frameworks can reliably catch these issues.