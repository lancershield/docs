# Token Migration Inconsistency

```YAML
id: TBA
title: Token Migration Inconsistency 
severity: H
category: migration
language: solidity
blockchain: [ethereum]
impact: User funds may be lost or duplicated during upgrade/migration
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-641
swc: SWC-135

```

## 📝 Description

- Token migration inconsistency occurs when transitioning from an old token contract to a new one without ensuring one-to-one accounting, including:
- Missing checks on prior migrations,
- Incorrect balance transfers,
- Inconsistent total supply mint/burn logic.
- This can lead to:
- Inflated or deflated supply in the new token
- Users claiming multiple times 
- Losing access to unclaimed balances.

## 🚨 Vulnerable Code

```solidity
contract TokenV2 is ERC20 {
    IERC20 public oldToken;

    function migrate(uint256 amount) external {
        // ❌ No migration state tracking, no old token burn
        oldToken.transferFrom(msg.sender, address(this), amount);
        _mint(msg.sender, amount);
    }
}

```

## 🧪 Exploit Scenario

Step-by-step impact:

1. A user calls migrate() with 100 tokens.
2. The contract transfers the tokens to itself but does not burn or track the migration.
3. The same user calls migrate() again with same approve() allowance.
4. They now hold 200 new tokens without losing the original 100 — double-counting occurs.

**Assumptions:**

- Old token isn't burned or locked.
- No migrated[msg.sender] guard or nonce check.
- Migration is open and unprotected.

## ✅ Fixed Code

``` solidity

contract TokenV2 is ERC20 {
    IERC20 public oldToken;
    mapping(address => bool) public migrated;

    function migrate(uint256 amount) external {
        require(!migrated[msg.sender], "Already migrated");
        require(amount > 0, "Invalid amount");

        oldToken.transferFrom(msg.sender, address(this), amount);
        migrated[msg.sender] = true;

        _mint(msg.sender, amount); // ✅ One-time mint
    }
}

```
Or with burn:

```solidity

function migrate(uint256 amount) external {
    oldToken.transferFrom(msg.sender, address(this), amount);
    IERC20Burnable(address(oldToken)).burn(amount);
    _mint(msg.sender, amount);
}

```


## 🛡️ Prevention

### Primary Defenses

- Track migration status per user (e.g., mapping(address => bool) or nonce).
- Ensure old tokens are burned, locked, or invalidated after migration.
-Implement a one-to-one supply accounting system with total migrated stats.

### Additional Safeguards

- Use migration windows and governance locks to stop duplicate attempts.
- Audit total supply comparisons: oldTotal == newTotal after migration period.
- If possible, pause old token transfers post-migration.

### Detection Methods

- Slither: migration-untracked, duplicate-mint, nonburned-old-token detectors.
- Manual audits comparing oldToken.transferFrom() + newToken._mint() paths.
- Test migration logic with edge cases (replay, partial balance, zero amounts).

## 🕰️ Historical Incidents

- **Name:** YAM Token Migration Issue 
- **Date:** 2020 
- **Impact:** Inconsistent migration caused supply bugs and governance issues 
- **Post-mortem:** [Link](https://medium.com/@yamfinance) 


## 📚 Further Reading

- [SWC-135: Inefficient or Unexpected Logic](https://swcregistry.io/docs/SWC-135) 
- [OpenZeppelin Upgradeable Token Docs](https://docs.openzeppelin.com/upgrades-plugins) 
- [Safe Token Upgrade/Migration Patterns](https://ethereum.org/en/developers/docs/standards/tokens/) 


---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Token Migration Inconsistency 
severity: H
score:
impact: 5         
exploitability: 3 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.0
```


---

## 📄 Justifications & Analysis

- **Impact**: High — mint/burn mismatch leads to inflation, loss, or trust issues.
- **Exploitability**: Attackers can repeat migration if unguarded.
- **Reachability**: Frequently occurs in airdrop-style migrations and token v2 launches.
- **Complexity**: Often introduced by missing just 1–2 lines of logic.
- **Detectability**: Detectable via tool-based and manual audits.