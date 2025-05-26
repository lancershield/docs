# Unrestricted Withdrawals

```YAML
id: TBA
title: Unrestricted Withdrawals
baseSeverity: C
category: access-control
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Unauthorized access to funds or assets
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-862
swc: SWC-105
```

## 📝 Description

- Unrestricted Withdrawals occur when a smart contract allows any user to call a withdrawal function that transfers funds or tokens, without properly verifying ownership, role, or deposit history. These flaws commonly arise from:
- Missing require() conditions (e.g., msg.sender == owner)
- Misused public or external visibility on privileged functions
- Incorrect state tracking of user balances
- Global withdrawals callable by anyone regardless of deposits
- This leads to complete loss of protocol assets, enabling malicious actors to withdraw user funds or protocol reserves without authorization.

## 🚨 Vulnerable Code

```solidity

contract Vault {
    function withdraw() public {
        payable(msg.sender).transfer(address(this).balance); // ❌ no ownership or deposit check
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A poorly coded staking or vault contract contains a withdraw() function without access checks.
2. An attacker notices the missing verification (e.g., balances[msg.sender], onlyOwner).
3. They call withdraw() directly.
4. The contract blindly transfers its entire balance to msg.sender.
5. All user deposits or reserves are stolen.

**Assumptions:**

- The contract holds ETH or ERC20 tokens.
- There is no require() verifying user deposit, stake, role, or identity.
- The withdraw() function is callable by the public.
- There are no per-user balance mappings or they are unused in withdrawal logic.

## ✅ Fixed Code

```solidity

mapping(address => uint256) public balances;

function withdraw() external {
    uint256 amount = balances[msg.sender];
    require(amount > 0, "No balance");
    balances[msg.sender] = 0;
    payable(msg.sender).transfer(amount);
}
```

## 🧭 Contextual Severity

```yaml
- context: "DeFi vault or fundraising contract with open withdrawals"
  severity: C
  reasoning: "Allows anyone to drain protocol funds immediately."
- context: "Owner-only function missing modifier"
  severity: H
  reasoning: "High severity if deployed and callable on mainnet."
- context: "Function unused or unreachable due to constructor logic"
  severity: L
  reasoning: "Lower severity if dead code or access unreachable."
```

## 🛡️ Prevention

### Primary Defenses

- Always enforce access control via require() statements tied to msg.sender.
- Track user deposits and match withdrawal limits accordingly.
- Apply onlyOwner or onlyRole modifiers to admin-level withdrawals.

### Additional Safeguards

- Integrate nonReentrant guards when transferring value.
- Use pullPayment pattern or withdraw queues to reduce abuse risk.
- Simulate misbehavior during testnet and fuzz scenarios.

### Detection Methods

- Slither: unprotected-function, missing-zero-reset, unchecked-send
- Manual review for functions transferring ETH, ERC20, or NFTs
- Fuzz testing with unauthorized users to simulate withdrawal attempts

## 🕰️ Historical Exploits

- **Name:** dForce Lendf.Me Hack 
- **Date:** 2020-04 
- **Loss:** ~$25,000,000 
- **Post-mortem:** [Link to post-mortem](https://quantstamp.com/blog/how-the-dforce-hacker-used-reentrancy-to-steal-25-million) 
- **Name:** Eminence (EMN) Exploit 
- **Date:** 2020-09 
- **Loss:** ~$15,000,000 
- **Post-mortem:** [Link to post-mortem](https://sampriestley.com/defi-arbs-explained-15m-eminence-attack/) 
- **Name:** Meerkat Finance Rug Pull 
- **Date:** 2021-03 
- **Loss:** ~$31,000,000 
- **Post-mortem:** [Link to post-mortem](https://coinmarketcap.com/academy/article/31m-stolen-after-meerkat-finance-launch-goes-wrong)
  
## 📚 Further Reading

- [SWC-105: Unprotected Function](https://swcregistry.io/docs/SWC-105) 
- [CWE-862: Missing Authorization](https://cwe.mitre.org/data/definitions/862.html) 
- [OpenZeppelin – ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) 
- [Slither Static Analyzer](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Unrestricted Withdrawals
severity: C
score:
impact: 5       
exploitability: 4 
reachability: 4  
complexity: 1    
detectability: 4 
finalScore: 4.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Immediate and irreversible asset loss.
- **Exploitability**: Requires only a public call to the flawed withdrawal function.
- **Reachability**: Occurs in many staking/farming contracts or miswritten vaults.
- **Complexity**: Very low—no technical sophistication required.
- **Detectability**: Easily found with tools like Slither or basic manual audit.
