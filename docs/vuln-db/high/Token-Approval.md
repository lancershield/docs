# Token Approval Without Spending Limits 

```YAML

id: TBA
title: Token Approval Without Spending Limits 
severity: H
category: token-approval
language: solidity
blockchain: [ethereum]
impact: Attacker can drain entire token balances if an approved spender is compromised
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<latest"]
cwe: CWE-275
swc: SWC-114

```

## 📝 Description

- Token approval without spending limits occurs when a user or contract approves another address (e.g., DEX, vault, spender) for the maximum `uint256` value (`2^256 - 1`), rather than a limited or session-specific amount. 
- This practice, while common for gas optimization, introduces a major risk:
- If the approved address is compromised, the attacker can:
- Drain all approved token balances at once,
- Front-run or race the intended use,
- Exploit long-lived approvals across unrelated sessions or dApps.
- This vulnerability has led to repeated losses in DeFi phishing attacks, compromised frontends, and wallet integrations.

## 🚨 Vulnerable Code

```solidity
function approveUnlimited(IERC20 token, address spender) external {
    token.approve(spender, type(uint256).max); // ❌ Dangerous unlimited approval
}

```

## 🧪 Exploit Scenario

Step-by-step attack:

1. A user approves a DEX contract with 2^256 - 1 allowance.
2. Later, that DEX's contract is compromised or a malicious upgrade is pushed.
3. The attacker calls transferFrom(user, attacker, entire balance) using the lingering allowance.
4. User’s tokens are drained instantly, with no further signature or interaction required.

**Assumptions:**

- The user or contract gives unlimited allowance.
- The spender is compromised or malicious in the future.

## ✅ Fixed Code

```solidity

function safeApprove(IERC20 token, address spender, uint256 amount) external {
    require(amount > 0 && amount <= token.balanceOf(msg.sender), "Invalid amount");
    token.approve(spender, amount); // ✅ Approve only what’s needed
}

```


## 🛡️ Prevention

### Primary Defenses

- Avoid approve(spender, type(uint256).max) unless absolutely required and trusted.
- Approve only the amount needed for a specific transaction.
- Use ERC-20 permit() (EIP-2612) for gasless, single-use approvals where possible.

### Additional Safeguards

- Implement approval race mitigation by setting allowance to zero before updating.
- Add expiration deadlines or nonces to time-limit approvals.
- Track and revoke stale or unused allowances in off-chain monitoring systems (e.g., revoke.cash).

### Detection Methods

- Slither: unlimited-approval, unsafe-token-approve, approve-max detectors.
- Manual audit of all approve(...) logic in contracts and dApp integration flows.
- Integration testing to ensure minimum necessary approvals are enforced.

## 🕰️ Historical Incidents

- **Name:** bZx Phishing Wallet Drainer 
- **Date:** 2021 
- **Impact:** >$55M drained from users who had unlimited approvals to compromised wallet
- **Post-mortem:** [Link](https://rekt.news/bzx-rekt/) 

## 📚 Further Reading

- [SWC-114: Unrestricted Token Approval](https://swcregistry.io/docs/SWC-114) 
- [EIP-2612 Permit Standard](https://eips.ethereum.org/EIPS/eip-2612) 
- [OpenZeppelin Docs – Approval Patterns](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20) 
- [Revoke.cash – Approval Risks](https://revoke.cash/learn) 

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Token Approval Without Spending Limits 
severity: H
score:
impact: 5         
exploitability: 3 
reachability: 4   
complexity: 1     
detectability: 5  
finalScore: 4.1


```


---

## 📄 Justifications & Analysis

- **Impact**: High — full balance loss is possible with one bad approval.
- **Exploitability**: Depends on spender being compromised or malicious, which is common in phishing.
- **Reachability**: Very common in DeFi frontends and wallet interactions.
- **Complexity**: Trivial fix — just approve the needed amount.
- **Detectability**: High — easily spotted in audits and flagged by tools.