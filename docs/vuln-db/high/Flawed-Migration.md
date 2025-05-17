# Flawed Migration Contracts

```YAML
id: TBA
title: Flawed Migration Contracts 
severity: H
category: migration
language: solidity
blockchain: [ethereum]
impact: Users can lose funds or mint duplicate tokens during token/system upgrades
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-640
swc: SWC-135
```

## 📝 Description

- Flawed migration contracts refer to unsafe upgrade/migration logic used when moving from one token, staking contract, or protocol version to another. 
- If migration contracts are poorly designed, they may:
- Allow multiple migrations for the same user,
- Fail to track migration status or amounts properly,
- Introduce supply mismatches due to incorrect mint/burn logic
- Be vulnerable to reentrancy or replay attacks.
- Migration contracts are especially risky if they combine minting, user-supplied approvals, and off-chain indexed logic (like Merkle roots) without robust validation.

## 🚨 Vulnerable Code

```solidity
contract BrokenMigrator {
    mapping(address => bool) public migrated;

    function migrate(address user, uint256 amount) external {
        // ❌ Does not check if amount has already been migrated
        oldToken.transferFrom(user, address(this), amount);
        _mint(user, amount);
    }
}
```

## 🧪 Exploit Scenario

Step-by-step impact:

1. A user migrates 100 tokens using the flawed migrate() function.
2. The contract does not mark the user or amount as migrated.
3. The same user (or attacker) repeats the call with the same allowance.
4. New tokens are minted again, doubling the balance without burning old tokens.

**Assumptions:**

- No tracking of per-user migration status or per-amount claim.
- Minting logic is tied only to transferFrom() without verification.

## ✅ Fixed Code

```solidity

contract SafeMigrator {
    mapping(address => uint256) public migratedAmount;

    function migrate(address user, uint256 amount) external {
        require(amount > 0, "Zero migration");
        require(migratedAmount[user] == 0, "Already migrated");

        oldToken.transferFrom(user, address(this), amount);
        migratedAmount[user] = amount;
        _mint(user, amount);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Track per-user claim or migration status with mappings (hasClaimed, migratedAmount, etc.).
- Use one-time Merkle proofs for scalable off-chain tracking with on-chain verification.
- Burn or lock migrated tokens to avoid dual liquidity and supply inflation.

### Additional Safeguards

- Use nonReentrant modifier if external calls are made during migration.
- Emit Migrated(address, amount) events for off-chain indexing.
- Include total migrated accounting to reconcile old vs new supply.

### Detection Methods

- Slither: migration-untracked, double-mint, missing-claim-status detectors.
- Manual audits of minting and migration logic with user-supplied inputs.
- Integration and fuzz tests simulating repeated migration attempts.

## 🕰️ Historical Exploits

- **Name:** KiloEx TrustedForwarder Exploit 
- **Date:** April 13, 2025 
- **Loss:** Approximately $7 million 
- **Post-mortem:** [Link to post-mortem](https://crypto.news/kiloex-reveals-7m-smart-contract-exploit-in-post-mortem-report/) 

## 📚 Further Reading

- [SWC-135: Code With No Effects / Unsafe Migration Logic](https://swcregistry.io/docs/SWC-135) 
- [Solidity Docs – Upgrade Patterns](https://docs.soliditylang.org/en/latest/) 
- [OpenZeppelin Safe Token Migration Guide](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Flawed Migration Contracts 
severity: H
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

- **Impact**: High — duplicate claims or wrong balances undermine protocol integrity.
- **Exploitability**: Medium — requires flawed checks, but no signature needed.
- **Reachability**: Found in a large number of token/staking upgrade contracts.
- **Complexity**: Easy to introduce; mitigated with proper state tracking.
- **Detectability**: High — visible with tests, audit rules, and user simulations.

