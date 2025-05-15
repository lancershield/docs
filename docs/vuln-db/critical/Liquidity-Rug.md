# Liquidity Rug Allowing Unauthorized 

```YAML

id: TBA
title: Liquidity Rug Allowing Unauthorized 
severity: C
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
- The contract owner, deployer, or privileged account **can remove all or a large portion of the pooled assets**, often without user consent or transparency. 
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
- **Date:** 2021 
- **Loss:** ~$3.4M 
- **Post-mortem:** [https://coinmarketcap.com/alexandria/article/squid-game-token-scam](https://coinmarketcap.com/alexandria/article/squid-game-token-scam) 


### 📚 Further Reading

- [SWC-100: Function Default Visibility](https://swcregistry.io/docs/SWC-100) 
- [DeFi Rug Pull Classification - Rekt](https://rekt.news/rugged/) 
- [OpenZeppelin AccessControl](https://docs.openzeppelin.com/contracts/4.x/access-control) 


---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Liquidity Rug Allowing Unauthorized 
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