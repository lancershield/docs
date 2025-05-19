# Admin Key Misuse

```YAML
id: TBA
title: Admin Key Misuse 
severity: C
category: access-control
language: solidity
blockchain: [ethereum]
impact: Contract owner can maliciously or accidentally seize funds or alter core behavior
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.4.0", "<latest"]
cwe: CWE-732
swc: SWC-105
```

## 📝 Description

- Admin key misuse occurs when privileged accounts (commonly `owner`, `admin`, or `governance`) retain unchecked or overly broad control over critical contract functions such as:
- Minting tokens.
- Transferring user funds.
- Changing logic contracts or configurations.
- Pausing/unpausing.
- Whitelisting arbitrary addresses.


## 🚨 Vulnerable Code

```solidity
contract AdminMint {
    address public owner;

    function mint(address to, uint256 amount) external {
        require(msg.sender == owner, "Not admin");
        _mint(to, amount); // ❌ No limits, time lock, or multisig
    }
}
```

## 🧪 Exploit Scenario

Step-by-step risk:

1. The contract deployer retains full owner control.
2. There are no constraints on minting, burning, transferring funds, or upgrades.
3. If the admin key is compromised or misused.
4. Unlimited tokens can be minted.
5. Funds can be redirected from users.
6. System logic can be maliciously upgraded.

**Assumptions:**

- Admin role is not protected by multisig, timelock, DAO governance, or transparency tooling.
- Critical functions are gated solely by onlyOwner or similar logic.

## ✅ Fixed Code

```solidity

contract SecureAdmin is Ownable {
    address public timelock;

    modifier onlyTimelock() {
        require(msg.sender == timelock, "Only timelock");
        _;
    }

    function mint(address to, uint256 amount) external onlyTimelock {
        require(amount < MAX_SUPPLY, "Exceeds cap");
        _mint(to, amount);
    }

    function transferOwnership(address newOwner) public override onlyOwner {
        require(newOwner != address(0), "Invalid");
        super.transferOwnership(newOwner); // ✅ Can be handed off to DAO or Gnosis Safe
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Replace raw admin roles with timelocks, multisigs, or governance modules.
- Limit admin powers:
- Cap mint amounts,
- Allow only upgrades through authorizeUpgrade() with role checks,
- Require DAO proposals or quorum.
- Clearly disclose admin capabilities in documentation.

### Additional Safeguards

- Use on-chain governance frameworks (e.g., OpenZeppelin Governor, Compound Governor Bravo).
- Emit events for all admin actions (AdminAction(type, caller, params)).
- Include "escape hatch" or renounceOwnership() functions for full decentralization.

### Detection Methods

- Slither: unprotected-admin, owner-unrestricted, no-multisig, centralization-risk detectors.
- Manual review of onlyOwner, msg.sender == admin, and unrestricted privileged flows.
- Governance simulation tests and audits.

## 🕰️ Historical Incidents

- **Name:** bZx Protocol Admin Key Compromise 
- **Date:** 2021 
- **Impact:** Approximately $55 million drained after attacker gained access to admin key 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/bzx-rekt/) 


## 📚 Further Reading

- [SWC-105: Unprotected Critical Function – SWC Registry](https://swcregistry.io/docs/SWC-105/) 
- [OpenZeppelin AccessControl Documentation](https://docs.openzeppelin.com/contracts/4.x/api/access)
- [OpenZeppelin Governor – DAO Governance Documentation](https://docs.openzeppelin.com/contracts/4.x/api/governance) 
- [Slither: Static Analyzer for Solidity](https://github.com/crytic/slither) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Admin Key Misuse 
severity: C
score:
impact: 5         
exploitability: 3 
reachability: 5   
complexity: 1     
detectability: 5 
finalScore: 4.2

```

---

## 📄 Justifications & Analysis

- **Impact**: Critical — full admin access enables any malicious behavior.
- **Exploitability**: Moderate — depends on insider behavior or key leakage.
- **Reachability**: High — most contracts include privileged functions.
- **Complexity**: Low — risks arise from overuse or lack of role segmentation.
- **Detectability**: High — tools and audits catch it easily.