# Flawed Slashing Conditions

```YAML

id: TBA
title: Flawed Slashing Conditions 
severity: H
category: consensus-mechanism
language: solidity
blockchain: [ethereum]
impact: Honest participants may be punished or malicious actors may evade slashing
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.6.0", "<latest"]
cwe: CWE-697
swc: SWC-132

```

## 📝 Description

- Flawed slashing conditions arise when the logic responsible for penalizing validators, stakers, or participants:
- Incorrectly identifies misbehavior,Fails to detect actual violations, Can be abused to penalize honest actors or evaded by malicious ones.
- This typically affects staking protocols, validator networks, or oracles, and can result in:
- Unfair slashing of good actors (loss of funds, reputation),No punishment for actual malicious behavior (downtime, equivocation, delay),Governance disputes and loss of network trust.

## 🚨 Vulnerable Code

```solidity
function slash(address validator, bool evidence) external onlySlasher {
    if (evidence) {
        stake[validator] = 0; // ❌ No validation of evidence, no protection
        emit Slashed(validator);
    }
}

```

## 🧪 Exploit Scenario

Step-by-step breakdown:

1. A malicious actor gains access to the slasher role (e.g. governance, multisig).
2. They pass a true flag as evidence, regardless of validator behavior.
3. The validator’s stake is immediately zeroed out, despite being honest.
4. Meanwhile, another malicious validator avoids slashing due to missing checks or poor liveness detection.

**Assumptions:**

- Validators avoid slashing by exiting before finalization.
- Network latency causes false downtime slashing.
- Lack of cryptographic proof in misbehavior evidence.

## ✅ Fixed Code

```solidity

function slash(address validator, bytes calldata evidence) external onlySlasher {
    require(validateEvidence(evidence, validator), "Invalid or insufficient proof");
    require(block.timestamp < evidenceExpiry[evidence], "Evidence expired");

    uint256 penalty = (stake[validator] * penaltyRate) / 100;
    stake[validator] -= penalty;
    emit Slashed(validator, penalty);
}

function validateEvidence(bytes calldata data, address validator) internal view returns (bool) {
    // e.g. double-signature, equivocation proof, signed block hash
    // ✅ Use verifiable cryptographic checks
    return isValidProof(data, validator);
}

```


## 🛡️ Prevention

### Primary Defenses

- Require cryptographic proof of misbehavior (e.g., BLS signatures, PoS equivocation).
- Enforce time bounds, cooldowns, and challenge periods before slashing.
- Ensure only authorized, auditable sources (e.g. consensus or quorum) can initiate slashing.

### Additional Safeguards

- Emit detailed Slashed logs with reason, evidence hash, and proof for on-chain audits.
- Allow appeal or dispute mechanisms for challenged slashing events.
- Use off-chain consensus proofs (e.g., attestation reports or ZK-based evidence) for accuracy.

### Detection Methods

- Slither: unconditional-slash, missing-evidence-check, slasher-role-danger detectors.
- Manual audit of all slashing conditions and roles.
- Simulation of false positive slashing attacks in fuzz tests.

##🕰️ Historical Exploits

- **Name:** Lido Node Operator Slashing Risk 
- **Date:** 2022 
- **Impact:** Concerns about centralized control over slashing logic
- **Post-mortem:** [https://lido.fi] 


## 📚 Further Reading

- [SWC-132: Incorrect Event Conditions](https://swcregistry.io/docs/SWC-132) 
- [CWE-697: Incorrect Comparison](https://cwe.mitre.org/data/definitions/697.html) 
- [Ethereum PoS Slashing Conditions](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/#slashing) 
- [Tendermint Slashing Logic](https://docs.tendermint.com/) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Flawed Slashing Conditions 
severity: H
score:
impact: 5         
exploitability: 3 
reachability: 3   
complexity: 4     
detectability: 4  
finalScore: 4.15


```


---

## 📄 Justifications & Analysis

- **Impact**: Critical — could unjustly destroy validator capital or let malicious actors go free.
- **Exploitability**: Moderate — needs privileged access or flawed logic.
- **Reachability**: Common in custom staking, L2, or oracle systems.
- **Complexity**: High — safe slashing systems require protocol-grade engineering.
- **Detectability**: High — flaws are exposed in audits and validator testing.