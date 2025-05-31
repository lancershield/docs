# EmissionScheduler 

```YAML
id: LS34M
title: EmissionScheduler 
baseSeverity: M
category: emission-logic
language: solidity
blockchain: [ethereum, arbitrum, optimism, polygon, bsc]
impact: Reward over-distribution, schedule bypass, or reward halt
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-668
swc: SWC-136
```

## 📝 Description

- An EmissionScheduler manages the timing and amount of token emissions over time. If poorly designed or misconfigured, it can lead to:
- Premature emissions: rewards released earlier than intended
- Excessive emissions: more tokens emitted than scheduled
- Reward halts: rewards stop due to block misalignment or missed schedule validation
- This flaw typically stems from:
- Inaccurate time or block comparison logic
- Missing upper/lower bounds for emission intervals
- Failure to synchronize emissions with claim logic

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract EmissionScheduler {
    uint256 public nextEmissionBlock;
    uint256 public emissionAmount = 1000 ether;
    uint256 public emissionInterval = 6500; // approx 1 day on Ethereum
    uint256 public lastEmissionBlock;

    function emitTokens(address to) external {
        require(block.number >= nextEmissionBlock, "Too early");
        lastEmissionBlock = block.number;
        nextEmissionBlock = lastEmissionBlock + emissionInterval;

        // ❌ No upper bound check or token cap
        // ❌ No access control
        _mint(to, emissionAmount);
    }

    function _mint(address to, uint256 amount) internal {
        // Logic omitted
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A developer implements a nextEmissionBlock mechanism to control when rewards are released.
2. Due to missing role checks, any external user can call emitTokens() as soon as block.number >= nextEmissionBlock.
3. The attacker monitors the block and calls the function the moment the threshold is met, triggering minting.
4. If the function is incorrectly structured or state is not correctly finalized, they can repeatedly call the function within the same block or exploit timing edge cases between emissions.
5. The attacker may drain multiple emissions ahead of schedule or monopolize the emission rewards, bypassing the intended pacing.

**Assumptions:**

- There is no global token emission cap or supply guardrail.
- block.number is used naively without handling re-entrancy or delayed execution properly.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/Ownable.sol";

contract SafeEmissionScheduler is Ownable {
    uint256 public nextEmissionBlock;
    uint256 public emissionAmount;
    uint256 public emissionInterval;
    uint256 public emissionCap;
    uint256 public totalEmitted;

    constructor(uint256 _startBlock, uint256 _interval, uint256 _cap, uint256 _amount) {
        nextEmissionBlock = _startBlock;
        emissionInterval = _interval;
        emissionCap = _cap;
        emissionAmount = _amount;
    }

    function emitTokens(address to) external onlyOwner {
        require(block.number >= nextEmissionBlock, "Emission not ready");
        require(totalEmitted + emissionAmount <= emissionCap, "Cap exceeded");

        nextEmissionBlock += emissionInterval;
        totalEmitted += emissionAmount;

        _mint(to, emissionAmount);
    }

    function _mint(address to, uint256 amount) internal {
        // Mint logic here
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Emission logic governs reward tokens or liquidity incentives"
  severity: M
  reasoning: "Over or under-rewarding users leads to economic imbalance"
- context: "Emission affects governance voting weight"
  severity: H
  reasoning: "Over-minting leads to takeover risks"
- context: "Internal-only or capped logic with governance gating"
  severity: L
  reasoning: "Low risk if emissions are restricted and bounded"
```

## 🛡️ Prevention

### Primary Defenses

- Lock emission control to roles or multisigs.
- Always enforce global cap (totalEmitted + amount <= max).
- Update state before transferring/minting tokens.

### Additional Safeguards

- Add block.number alignment checks to prevent re-entrant re-triggers.
- Use emitEvents() for emission transparency and auditability.
- Optionally batch emissions over multiple periods with smoothing logic.

### Detection Methods

- Static analysis for emission logic using block.number without checks
- Look for emission functions callable by unprivileged users
- Tools: Slither (timestamp-dependence, missing-authorization), Echidna tests

## 🕰️ Historical Exploits

- **Name:** Dynamic Set Dollar Emission Drift 
- **Date:** 2021-01 
- **Loss:** N/A (design flaw caused unpredictable emission pacing) 
- **Post-mortem:** [Link to post-mortem](https://www.reddit.com/r/defi/comments/ku8vps/dsd_broken_tokenomics/)
  
## 📚 Further Reading

- [SWC-136: Unrestricted Critical Variable Update](https://swcregistry.io/docs/SWC-136) 
- [Solidity Docs – Block Number Dependency](https://docs.soliditylang.org/en/latest/security-considerations.html#timestamp-dependence) 
- [OpenZeppelin Contracts – Timelock and Emission Control](https://docs.openzeppelin.com/contracts/4.x/governance#timelockcontroller)

--- 

## ✅ Vulnerability Report

```markdown
id: LS34M
title: EmissionScheduler 
severity: M
score:
impact: 4   
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 4  
finalScore: 4.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Can break tokenomics by releasing too many or too few tokens
- **Exploitability**: Without protection, any actor may drain the scheduler
- **Reachability**: Found in staking, liquidity mining, and emission vaults
- **Complexity**: Medium complexity with timing + arithmetic
- **Detectability**: Easy to detect with audits or simulation