# Zero Fee Manipulation

```YAML
id: TBA
title: Zero Fee Manipulation 
severity: M
category: fee-evasion
language: solidity
blockchain: [ethereum]
impact: Users can bypass intended fee logic, drain bandwidth, or access premium actions for free
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-285
swc: SWC-136
```

## 📝 Description

- Zero fee manipulation occurs when a protocol that intends to charge users for access, minting, trading, or claiming fails to properly enforce fee payment, allowing attackers or users to:
- Bypass intended transaction costs,Exploit fee exemptions or incorrectly handled `msg.value == 0` conditions.
- Access premium or limited functions for free,Conduct spam or DoS attacks due to underpriced operations.
- This flaw undermines protocol revenue, fairness, and on-chain economics — especially in mints, swaps, airdrops, or access-gated functions.

## 🚨 Vulnerable Code

```solidity
function mint() external payable {
    require(msg.value >= fee, "Fee too low");
    _mint(msg.sender); // ❌ no fallback if msg.value == 0 and fee == 0
}
```

## 🧪 Exploit Scenario

Step-by-step abuse:

1. A dApp enables public minting with a fee = 0.01 ETH.
2. The contract lacks proper enforcement or incorrectly handles exemptions.
3. An attacker sets up a contract to call mint() with msg.value = 0.
4. Due to unchecked edge conditions (e.g., exemption not validated), mint succeeds.
5. The attacker spams free mints, drains remaining supply, and resells on secondary markets.

**Assumptions:**

- An attacker exploits a poorly governed feeExempt flag to whitelist themselves.

## ✅ Fixed Code

```solidity

function mint() external payable {
    require(!feeExempt[msg.sender] || msg.value >= fee, "Invalid fee");
    _mint(msg.sender);
}
```

## 🛡️ Prevention

### Primary Defenses

- Enforce minimum msg.value with strict checks for all fee-requiring functions.
- Do not expose feeExempt modifiers without secure admin controls or public logging.
- Default fee logic should be opt-in only via verifiable whitelists, not opt-out.

### Additional Safeguards

- Emit fee-related events (FeePaid, FeeBypassed) for traceability.
- Protect fee logic with tests that include msg.value = 0, gas limits, and edge flags.
- Deny zero-fee interactions unless explicitly authorized via signature or Merkle proof.

### Detection Methods

- Slither: fee-evasion, msg.value-zero, insecure-exemption detectors.
- Fuzz testing for all payable functions with boundary conditions.
- Unit tests enforcing fee compliance across normal and exempted users.

## 🕰️ Historical Exploits

- **Name:** Mango Markets Exploit 
- **Date:** October 2022 
- **Loss:** $117 million 
- **Post-mortem:** [Link to post-mortem](https://cryptodamus.io/en/articles/news/mango-markets-hack-defi-s-100m-wake-up-call-legal-drama-lessons-learned) 

## 📚 Further Reading

- [SWC-136: Unintended Economic Conditions](https://swcregistry.io/docs/SWC-136) 
- [Solidity Docs – msg.value](https://docs.soliditylang.org/en/latest/) 
- [Fee Design in Token Protocols](https://ethereum.org/en/developers/docs/gas/) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Zero Fee Manipulation 
severity: M
score:
impact: 4         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 3.75

```

---

## 📄 Justifications & Analysis

- **Impact**: High — bypassed fees ruin supply control and cause revenue or fairness loss.
- **Exploitability**: Moderate — often public and can be automated by bots.
- **Reachability**: Common in payable functions or exemption patterns.
- **Complexity**: Low — a few misplaced conditionals often cause this.
- **Detectability**: High — easily caught with tools or test harnesses.