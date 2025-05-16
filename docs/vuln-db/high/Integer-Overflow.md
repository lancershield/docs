# Integer Overflow and Underflow

```YAML
id: TBA
title: Integer Overflow and Underflow in Arithmetic Operations
severity: H
category: arithmetic
language: solidity
blockchain: [ethereum]
impact: Incorrect state updates or fund miscalculations
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<0.8.0"]
cwe: CWE-190
swc: SWC-101
```

## 📝 Description

- Integer overflow and underflow occur when arithmetic operations exceed the maximum or minimum value of the data type. In Solidity versions prior to 0.8.0, arithmetic operations on `uint` or `int` types do not automatically revert on overflow or underflow. 
- This can be exploited to bypass balances, mint excess tokens, or cause denial of service via incorrect logic paths.

## 🚨 Vulnerable Code

```solidity
contract Token {
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient");
        balances[msg.sender] -= amount; // underflow if amount > balance
        balances[to] += amount;         // overflow possible here too
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker calls transfer() with amount > balances[msg.sender].
2. If overflow/underflow checks are not enforced, subtraction wraps around to a huge value.
3. balances[msg.sender] becomes very large; balances[to] also overflows.
4. Attacker gains an unintended balance or breaks accounting logic.

**Assumptions:**

- Contract is compiled with Solidity <0.8.0.
- No usage of SafeMath or manual overflow checks.

## ✅ Fixed Code

```solidity
// Solidity >=0.8.0 has built-in overflow checks
contract SafeToken {
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient");
        unchecked {
            balances[msg.sender] -= amount;
            balances[to] += amount;
        }
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use Solidity >=0.8.0, which includes built-in overflow/underflow checks.
- For older versions, use OpenZeppelin’s SafeMath library.
- Avoid unchecked arithmetic unless performance-justified and safe.

### Additional Safeguards

- Write invariant tests for balance and supply correctness.
- Perform fuzzing to explore large input edge cases.

### Detection Methods

- Slither: arithmetic, unprotected-arithmetic detectors.
- Mythril, Echidna: symbolic and fuzzing-based overflow detection.
- Manual audit of arithmetic hotspots (balance, supply, loop counters).

## 🕰️ Historical Exploits

- **Name:** Proof of Weak Hands Coin (PoWHCoin) Exploit 
- **Date:** 2018-03-09 
- **Loss:** Approximately 866 ETH 
- **Post-mortem:** [Link to post-mortem](https://101blockchains.com/integer-overflow-attacks/) 
- **Name:** BeautyChain (BEC) Token Exploit 
- **Date:** 2018-04-22 
- **Loss:** Over $6 million 
- **Post-mortem:** [Link to post-mortem](https://dreamlab.net/en/blog/post/ethereum-smart-contracts-vulnerabilities-integer-overflow-and-underflow/) 
  

## 📚 Further Reading

- [SWC-101: Integer Overflow and Underflow](https://swcregistry.io/docs/SWC-101) 
- [OpenZeppelin – SafeMath Documentation](https://docs.openzeppelin.com/contracts/2.x/api/math) 
- [Solidity Docs – Arithmetic Safety in >=0.8.0](https://docs.soliditylang.org/en/v0.8.0/control-structures.html#checked-or-unchecked-arithmetic) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Integer Overflow and Underflow in Arithmetic Operations
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.3
```

---

## 📄 Justifications & Analysis

- **Impact**: Incorrect balances can lead to financial loss, infinite token minting, or denial of service.
- **Exploitability**: Easy in pre-0.8.0 contracts without SafeMath; requires only a crafted input.
- **Reachability**: Affects core logic like transfers, staking, accounting.
- **Complexity**: Attack is input-only; no contract interaction needed.
- **Detectability**: Slither, Mythril, and static analyzers reliably catch this if not suppressed.
