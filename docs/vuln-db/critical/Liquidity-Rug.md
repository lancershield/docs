# Liquidity Rug  

```YAML
id: LS05C
title: Liquidity Rug  
baseSeverity: C
category: access-control
language: solidity
blockchain: [ethereum]
impact: Protocol liquidity can be drained suddenly, leaving users unable to trade or withdraw
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.5.0", "<latest"]
cwe: CWE-285
swc: SWC-100
```

## 📝 Description

- Liquidity rug refers to a scenario in decentralized finance (DeFi) where liquidity providers or users deposit tokens into a protocol (e.g., DEX, staking contract, vault).
- The contract owner, deployer, or privileged account can remove all or a large portion of the pooled assets, often without user consent or transparency. 
- Loss of funds or token value collapse,Broken trading pairs and stuck LP tokens, Destroyed user trust and ecosystem credibility.

## 🚨 Vulnerable Code

```solidity
contract LPVault {
    address public owner;
    IERC20 public lpToken;

    function emergencyWithdraw() external {
        require(msg.sender == owner, "Not authorized");
        lpToken.transfer(owner, lpToken.balanceOf(address(this))); // ❌ Entire pool drained by admin
    }
}
```

## 🧪 Exploit Scenario

Step-by-step:

1. Users deposit LP tokens into a staking or farming contract to earn rewards.
2. Owner calls emergencyWithdraw() without user notification or restriction.
3. The contract transfers 100% of the LP tokens to the owner.
4. Users attempting to withdraw now face zero liquidity and cannot recover funds.

**Assumptions:**

- The smart contract holds user-deposited liquidity (e.g., LP tokens, stablecoins, or staking assets).
- There exists an admin-controlled function (e.g., withdraw(), emergencyWithdraw(), sweep()) .
- Allows transfer of all or a significant portion of pooled tokens, and
- Is either callable by the deployer/owner/multisig without user consent or governance delay.

## ✅ Fixed Code

```solidity

function emergencyWithdraw() external onlyGovernance {
    require(block.timestamp > emergencyUnlockTime, "Emergency not active");
    uint256 ownerShare = lpToken.balanceOf(address(this)) / 10; // ✅ Limited withdrawal
    lpToken.transfer(governance, ownerShare);
}
```

## 🧭 Contextual Severity

```yaml
- context: "Retail-facing token with centralized liquidity control"
  severity: C
  reasoning: "Owner can remove liquidity at any time, causing total loss for users."
- context: "Liquidity burned or locked in time-locked contract"
  severity: L
  reasoning: "Risk significantly reduced if proper lock mechanism is in place."
- context: "DAO-controlled liquidity pool"
  severity: M
  reasoning: "DAO may still vote to rug, but it's visible and time-delayed."
```

## 🛡️ Prevention

### Primary Defenses

- Never allow unrestricted admin access to pooled liquidity.
-  DAO-based withdrawal approval or timelocks for emergency functions.
- Cap emergency withdrawals (e.g., max 10–20% of total pool).

### Additional Safeguards

- Emit EmergencyWithdrawalTriggered events with reason and governance signature.
- Implement governance delay via contracts like TimelockController.
- Disclose all privileged functions in UI and audits.

### Detection Methods

- Slither: access-control-unrestricted, admin-sweep, emergency-withdraw detectors.
- Manual code audit of functions with transfer, transferFrom, sweep, or withdrawAll.
- Look for use of msg.sender == owner with no further restrictions.

## 🕰️ Historical Exploits

- **Name:** Squid Game Token Rug Pull 
- **Date:** 2021-11-01
- **Loss:** Approximately $3.38 million 
- **Post-mortem:** [Link to post-mortem](https://www.coindesk.com/markets/2021/11/01/squid-game-token-crashes-developers-say-theyve-left-the-project) 
- **Name:** AnubisDAO Rug Pull 
- **Date:** 2021-10-29
- **Loss:** Approximately $60 million 
- **Post-mortem:** [Link to post-mortem](https://www.binance.com/en/square/post/12028514221577) 
  

## 📚 Further Reading

- [SWC-136: Unexpected Behavior from External Calls – SWC Registry](https://swcregistry.io/docs/SWC-136/) 
- [Chainalysis: 2021 Crypto Scam Revenues](https://www.chainalysis.com/blog/2021-crypto-scam-revenues/) 
- [Cointelegraph: What is a rug pull in crypto and 6 ways to spot it?](https://cointelegraph.com/explained/crypto-rug-pulls-what-is-a-rug-pull-in-crypto-and-6-ways-to-spot-it) 
- [SlowMist: Cryptocurrency Scams Unveiled – Insights and Prevention](https://slowmist.medium.com/cryptocurrency-scams-unveiled-insights-and-prevention-800d9dc1b0f1) 
  
---

## ✅ Vulnerability Report 

```markdown
id: LS05C
title: Liquidity Rug  
severity: C
score:
impact: 5         
exploitability: 3 
reachability: 4  
complexity: 2     
detectability: 5  
finalScore: 4.1
```

---

## 📄 Justifications & Analysis

- **Impact**: Critical — results in complete fund drain for users.
- **Exploitability**: Moderate — relies on admin control or owner key.
- **Reachability**: Common in small or unaudited DeFi protocols.
- **Complexity**: Low — often a single function call.
- **Detectability**: High — easily identified via static tools or manual review.