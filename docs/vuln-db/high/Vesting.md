# Vesting Shortcut Exploits 

```YAML

id: TBA
title: Vesting Shortcut Exploits 
severity: H
category: tokenomics
language: solidity
blockchain: [ethereum]
impact: Users may bypass vesting schedules and claim unearned tokens
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-863
swc: SWC-124

```

## 📝 Description

- Vesting shortcut exploits occur when users are able to bypass time-locked or vesting conditions due to flawed logic in the vesting or claiming contract. 
- This may happen due to: Unchecked edge conditions in `claim()` or `release()` functions,Misused block timestamps or assumptions around `block.number`.
- Mistakes in the linear vesting formula or cliff handling.
- As a result, malicious or impatient users can claim more tokens than earned, drain the vesting pool prematurely, or invalidate fair distribution mechanisms.

## 🚨 Vulnerable Code

```solidity
function claim() public {
    uint256 amount = totalVested[msg.sender]; // ❌ does not factor in vesting schedule
    require(amount > 0, "Nothing to claim");
    totalVested[msg.sender] = 0;
    token.transfer(msg.sender, amount);
}

```


## 🧪 Exploit Scenario

Step-by-step bypass:

1. A user is assigned a 1-year vesting schedule with linear release.
2. The claim() function mistakenly transfers the full totalVested balance without checking the current vesting time.
3. The user calls claim() immediately after allocation.
4. They receive 100% of their tokens, bypassing the vesting cliff and violating tokenomics.

**Assumptions:**

- Cliff ignored: Cliff date check missing or set in the past.
- Multiplying by block.timestamp instead of using time deltas.
- Floating-point arithmetic errors causing overflows in earned().

## ✅ Fixed Code

```solidity

function claim() external {
    uint256 releasable = vestedAmount(msg.sender) - claimed[msg.sender];
    require(releasable > 0, "Nothing vested yet");

    claimed[msg.sender] += releasable;
    token.transfer(msg.sender, releasable);
}

function vestedAmount(address user) public view returns (uint256) {
    if (block.timestamp < startTime[user] + cliff) return 0;
    uint256 elapsed = block.timestamp - startTime[user];
    if (elapsed >= duration[user]) return totalVested[user];

    return (totalVested[user] * elapsed) / duration[user];
}

```


## 🛡️ Prevention

### Primary Defenses

- Use elapsed time checks to compute release amounts (now - startTime).
- Enforce cliff logic before any claim is allowed.
- Track claimed amount separately to prevent reclaims.

### Additional Safeguards

- Use battle-tested vesting libraries (e.g., OpenZeppelin TokenVesting.sol).
- Avoid floating-point math or division truncation bugs.
- Test vesting schedules over edge conditions (cliff, near expiry, before start).

### Detection Methods

- Slither: vesting-bypass, missing-cliff-check, timestamp-math-bug detectors.
- Unit tests for multiple time checkpoints and boundary conditions.
- Manual audit of claim() and vestedAmount() for time logic correctness.

## 🕰️ Historical Incidents

- **Name:** Liquity Vesting Bug 
- **Date:** 2021 
- **Impact:** Users could claim full tokens before the vesting cliff 
- **Post-mortem:** [Link](https://liquity.org) 


## 📚 Further Reading

- [SWC-124: Missing Time/Condition Check](https://swcregistry.io/docs/SWC-124) 
- [OpenZeppelin Token Vesting Contract](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#TokenVesting) 
- [Best Practices for Vesting](https://ethereum.org/en/developers/docs/standards/tokens/vesting/) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Vesting Shortcut Exploits 
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

- **Impact**: Critical in token launches — can ruin distribution or allow insider abuse.
- **Exploitability**: Moderate — no external access needed if claim is public.
- **Reachability**: Often found in early-stage or custom-built vesting schedules.
- **Complexity**: Easy to fix once identified, but often overlooked.
- **Detectability**: High — test and audit coverage should always include time-based logic.