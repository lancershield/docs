# LP Token Burn 

```YAML
id: LS46H
title: LP Token Burn 
baseSeverity: H
category: liquidity
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Liquidity removed or destroyed, disabling swaps and affecting token price stability
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-284: Improper Access Control
swc: SWC-106: Unprotected Selfdestruct Instruction
```

## 📝 Description

- In DeFi systems, Liquidity Provider (LP) tokens represent shares in a liquidity pool (e.g., on Uniswap, PancakeSwap). These LP tokens can be:
- Burned to remove liquidity, or Locked in smart contracts to signal permanence and deter rugpulls.
- If LP tokens are not locked and are later burned manually or via contract, it results in:
- Irreversible destruction of liquidity
- Disabling of DEX swaps for that token pair, permanent price distortion or zero liquidity on-chain

## 🚨 Vulnerable Code

```solidity

IERC20 public lpToken;

function burnLiquidity() external onlyOwner {
    uint256 amount = lpToken.balanceOf(address(this));
    lpToken.transfer(address(0), amount); // ❌ sends LP to dead address
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A token project launches and provides $100,000 of liquidity to a DEX.
2. The LP tokens are sent to a contract wallet but are not time-locked or vested.
3. At any point, the owner calls burnLiquidity() to send LP tokens to the zero address (0x00...dead).
4. This removes all liquidity permanently, making swaps impossible.
5. Traders are left holding untradeable tokens; price discovery is broken.

**Assumptions:**

- LP tokens are held in an EOA or smart contract without a locking, vesting, or timelock mechanism.
- There is a function or ability to irreversibly burn LP tokens (via transfer to dead address).

## ✅ Fixed Code

```solidity

address public lpTimelock;
IERC20 public lpToken;

constructor(address _lpToken) {
    lpToken = IERC20(_lpToken);
    lpTimelock = address(new Timelock(msg.sender, 30 days)); // ✅ time-locked control
}

function burnLiquidity() external {
    require(msg.sender == lpTimelock, "Locked");
    // Execute only after timelock delay or governance vote
}
```

## 🧭 Contextual Severity

```yaml
- context: "Mainnet AMM or LP staking contract with public LP token burns"
  severity: H
  reasoning: "Severe fund mismanagement, price distortion, or locked value"
- context: "Private system with admin-controlled burns"
  severity: M
  reasoning: "Damage scope limited to internal operations"
- context: "LP token burn always accompanied by reserve sync or redemption"
  severity: I
  reasoning: "Properly mitigated"
```

## 🛡️ Prevention

### Primary Defenses

- Time-lock LP token custody with a trusted smart contract (e.g., Gnosis Safe, Timelock).
- Make burn irreversible only after delay, DAO vote, or under clear exit plan.
- Use LOCKED() proof (public variable or dashboard) for transparency.

### Additional Safeguards

- Publish LP token destination at launch.
- Prevent burning LP tokens during active trading periods.
- Educate users on verifying LP lock status before investing.

### Detection Methods

- Slither rule: detect transfer(0x0) or burn of LP token type
- Manual code review for LP ownership & lock logic
- Tools: RugDoc, TokenSniffer, GitHub lock verifiers

## 🕰️ Historical Exploits

- **Name**: SafeMoon Burn Bug Exploit 
- **Date**: 2023-03-30 
- **Loss**: $8.9 
- **Post-mortem**: [Link to post-mortem](https://www.bitdefender.com/en-us/blog/hotforsecurity/hacker-exploits-safemoon-burn-bug-steals-8-9-million-from-liquidity-pool) 
- **Name**: ShadowFi Burn Function Exploit 
- **Date**: 2022-09-02 
- **Loss**: $301,000 
- **Post-mortem**: [Link to post-mortem](https://medium.com/quillhash/shadowfi-301k-burn-function-exploit-analysis-quillaudits-45a17ce04193) 
- **Name**: BBX Token Burn Mechanism Exploit 
- **Date**: 2025-03-20 
- **Loss**: $12,000 
- **Post-mortem**: [Link to post-mortem](https://blog.solidityscan.com/bbx-token-hack-analysis-f2e962c00ee5) 

## 📚 Further Reading

- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html) 
- [SWC-106: Unprotected Critical Function](https://swcregistry.io/docs/SWC-106) 
- [Timelock Contracts – OpenZeppelin](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController) 

---

## ✅ Vulnerability Report

```markdown
id: LS46H
title: LP Token Burn 
severity: H
score:
impact: 5  
exploitability: 4   
reachability: 4  
complexity: 1  
detectability: 4  
finalScore: 4.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Permanent loss of DEX liquidity makes the token untradable and undermines trust.
- **Exploitability**: Trivial if owner has LP access and no lock.
- **Reachability**: Found in many new or under-audited token projects.
- **Complexity**: Very simple — just a single transfer to dead address.
- **Detectability**: Easily detectable by checking LP token holder and timelock presence.

