# Arbitrary from in transferFrom

```YAML
id: TBA
title: Arbitrary from in transferFrom Leads to Unauthorized Transfers
severity: H
category: access-control
language: solidity
blockchain: [ethereum]
impact: Unauthorized token transfers from arbitrary addresses
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-862
swc: SWC-135
```

## 📝 Description

- This vulnerability occurs when a smart contract incorrectly uses the ERC-20 transferFrom() function without properly validating the from address.
- If the contract allows a user to arbitrarily pass any from address into the function, it may allow unauthorized or unintended transfers from addresses that the caller controls allowances for—or worse, that they do not.
- Typically, transferFrom() is used by contracts that act as intermediaries for token transfers (like staking, escrow, or marketplaces). 
- However, if the contract does not restrict who can call it or doesn’t validate that the msg.sender is authorized to act on behalf of the from address, it opens the door to privilege escalation or fund theft.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IERC20 {
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}

contract VulnerableEscrow {
    IERC20 public token;

    constructor(IERC20 _token) {
        token = _token;
    }

    function deposit(address from, uint256 amount) external {
        // ❌ No check on who is calling or if they are allowed to act for `from`
        token.transferFrom(from, address(this), amount);
    }
}
```

## 🧪 Exploit Scenario

1. User A approves 1,000 tokens for the VulnerableEscrow contract.
2. Attacker B calls deposit(UserA, 1_000) without permission from User A.
3. Tokens are pulled from User A’s wallet into the contract.
4. Attacker B could combine this with a withdrawal function to steal the funds.

**Assumptions:**

- from address is not validated against msg.sender.
- transferFrom is used as part of deposit or stake flow.
- Users unknowingly approve malicious or unguarded contracts.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

interface IERC20 {
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}

contract SafeEscrow {
    IERC20 public token;

    constructor(IERC20 _token) {
        token = _token;
    }

    function deposit(uint256 amount) external {
        // ✅ Always use msg.sender as the source
        token.transferFrom(msg.sender, address(this), amount);
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Only allow transferFrom(msg.sender, ...) unless explicitly authorized.
- Do not expose from address as a user-controlled input without validating ownership or permissions.

### Additional Safeguards

- Use role-based access control for privileged actions.
- Implement internal authorization (e.g., signature validation if delegation is intended).
- Use msg.sender as the token source by default.

### Detection Methods

- Scan for functions using transferFrom with external from input.
- Check for missing validation of msg.sender against the from address.
- Tools: Slither (arbitrary-from rule), MythX, manual audit

## 🕰️ Historical Exploits

- **Name:** Useless Ethereum Token (UET) `transferFrom` Exploit 
- **Date:** December 2017 
- **Loss:** Unauthorized token transfers 
- **Post-mortem:** [Link to post-mortem](https://cvefeed.io/vuln/detail/CVE-2018-10468)
  

## 📚 Further Reading

- [SWC-135: Incorrect Authorization](https://swcregistry.io/docs/SWC-135/)  
- [OpenZeppelin: ERC-20 Approval Best Practices](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-) 
- [Ethereum StackExchange: Misuse of transferFrom](https://ethereum.stackexchange.com/questions/80084/when-to-use-transfer-vs-transferfrom) 
  
---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Arbitrary from in transferFrom Leads to Unauthorized Transfers
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Allows third parties to pull tokens from user accounts without their active consent—often without full awareness.
- **Exploitability**: If allowance is set, any malicious or unaware contract can trigger it with no friction.
- **Reachability**: Function is public and often assumed to be safe due to ERC-20 standard.
- **Complexity**: Simple oversight; no advanced logic or math required.
- **Detectability**: Easy to miss unless explicitly validating who can call transferFrom on behalf of whom. Some static tools flag it; others may not.

