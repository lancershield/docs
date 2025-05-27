# Excessive Gas Refund Abuse 

```YAML
id: TBA
title: Excessive Gas Refund Abuse 
baseSeverity: M
category: gas-refund
language: solidity
blockchain: [ethereum, optimism, arbitrum, polygon, bsc]
impact: DoS via block gas exhaustion, fee manipulation, and economic disruption
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-770
swc: SWC-128
```

## 📝 Description

- Excessive gas refund abuse occurs when a smart contract is intentionally designed or manipulated to generate large gas refunds by self-destructing contracts or clearing storage slots. This refund can be exploited to:
- Bypass block gas limits (by reclaiming gas)
- Manipulate gas-based rewards or MEV strategies
- Launch DoS attacks by spamming transactions at low effective cost
- Although EIP-3529 reduced the refund cap, many EVM-compatible chains (or older forks) still allow partial gas refunds, making this attack vector relevant in certain environments or for strategic block manipulation.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract RefundGriefer {
    mapping(uint256 => uint256) public junk;

    function fillStorage(uint256 count) external {
        for (uint256 i = 0; i < count; i++) {
            junk[i] = 1;
        }
    }

    function clearStorage(uint256 count) external {
        for (uint256 i = 0; i < count; i++) {
            delete junk[i]; // ❌ triggers gas refund
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. An attacker creates a contract that stores thousands of key-value pairs in storage (e.g., junk[i] = 1).
2. They submit a second transaction that clears this storage using delete junk[i], which triggers gas refunds under the EVM refund model.
3. Because of the refunded gas, the effective gas cost is reduced, allowing them to submit multiple transactions that would normally exceed block gas limits.
4. The attacker loops this process or pairs it with high-cost logic to monopolize blockspace and execute MEV-heavy logic cheaply.
5. In some environments, this behavior enables transaction flooding, DoS, or fee distortion, especially in conjunction with flashbots or priority auctions.

**Assumptions:**

- The chain honors partial gas refunds (e.g., pre-EIP-3529 behavior or EVM forks).
- Refund-based gas cost reductions can affect block limits or validator economics.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeStorage {
    mapping(uint256 => uint256) private junk;

    function writeData(uint256 index, uint256 value) external {
        require(junk[index] == 0, "Already written");
        junk[index] = value;
    }

    // ❌ No delete function exposed to public
    // Admins must carefully manage gas refund paths via timelocked upgrades
}
```

## 🧭 Contextual Severity

```yaml
- context: "Gas refund used to game transaction fee systems or rewards"
  severity: M
  reasoning: "Unfair advantage, but no direct asset theft"
- context: "Used in conjunction with block stuffing or relayer DoS"
  severity: H
  reasoning: "May prevent user transactions or drain gas subsidy pool"
- context: "Refunds disabled (e.g., post-EIP-3529, no selfdestruct)"
  severity: I
  reasoning: "Abuse no longer viable in current gas model"
```

## 🛡️ Prevention

### Primary Defenses

- Avoid exposing public delete/clear/selfdestruct paths
- Limit use of storage-clearing patterns in user-facing flows

### Additional Safeguards

- Rate-limit gas refund paths
- Use off-chain simulations to analyze potential refund loops
- Upgrade to environments that enforce EIP-3529, which caps gas refunds

### Detection Methods

- Static analysis of loops calling delete, selfdestruct, or SSTORE with zero
- Look for gas-optimized constructs that may be used maliciously
- Tools: Slither (gas-cost, storage-delete), MythX, Tenderly profiler

## 🕰️ Historical Exploits

- **Name:** FTX Exchange Gas Abuse 
- **Date:** 2022-10-12 
- **Loss:** ~81 ETH in gas fees 
- **Post-mortem:** [Link to post-mortem](https://github.com/MikeSpa/ethereum-exploit) 
- **Name:** dYdX Gasless Deposit Exploit 
- **Date:** 2022-08-10 
- **Loss:** ~0.5 ETH per day 
- **Post-mortem:** [Link to post-mortem](https://medium.com/@hacxyk/stealing-gas-from-dydx-0-5-eth-a-day-712c5fdc43a3) 
  
## 📚 Further Reading

- [SWC-128: Gas Griefing](https://swcregistry.io/docs/SWC-128)
- [CWE-770: Allocation of Resources Without Limits](https://cwe.mitre.org/data/definitions/770.html)
- [EIP-3529: Reduction in Refunds](https://eips.ethereum.org/EIPS/eip-3529) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Excessive Gas Refund Abuse 
severity: M
score:
impact: 4   
exploitability: 4 
reachability: 3   
complexity: 3     
detectability: 4  
finalScore: 3.85
```

---

## 📄 Justifications & Analysis

- **Impact**: Can reduce transaction costs below expected levels or congest the chain
- **Exploitability**: Abuse is trivial if delete/selfdestruct paths are open
- **Reachability**: Requires exposed functions or malicious design
- **Complexity**: Technical, but well-understood among advanced MEV actors
- **Detectability**: Profiled easily through gas behavior in testnets and prod