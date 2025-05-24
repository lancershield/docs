# Token Whitelisting Flaws

```YAML
id: TBA
title: Token Whitelisting Flaws
severity: H
category: validation
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Unauthorized tokens accepted, leading to fund loss, exploit, or market manipulation
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-284
swc: SWC-135
```

## 📝 Description

- Token Whitelisting Flaws occur when protocols rely on a whitelist to determine which ERC20 tokens are allowed in specific functions (e.g., swaps, staking, rewards, vaults), but fail to enforce, validate, or update the whitelist securely. Common issues include:
- Tokens not being properly validated against the whitelist
- Tokens whitelisted but later replaced via proxies
- Tokens with unexpected transfer behavior (e.g., fee-on-transfer) that bypass assumptions
- Contracts using transferFrom() or transfer() without whitelist enforcement

## 🚨 Vulnerable Code

```solidity

mapping(address => bool) public isWhitelisted;

function deposit(address token, uint256 amount) external {
    // ❌ Missing check: isWhitelisted[token]
    IERC20(token).transferFrom(msg.sender, address(this), amount);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Protocol claims to only accept whitelisted tokens, but forgets to enforce require(isWhitelisted[token]).
2. Attacker deploys a malicious ERC20 token that always returns true for transferFrom() but doesn't actually transfer any tokens.
3. They call deposit() with their fake token.
4. Protocol records a deposit, triggering unearned rewards or share issuance.
5. Attacker calls withdraw() or claim(), receiving real assets (e.g., ETH, USDC).

**Assumptions:**

- The protocol uses whitelists for accepted tokens but does not enforce them at the point of usage.
- The attacker can supply custom or malicious tokens with ERC20 interfaces.
- There is no validation of actual transfer success or balance deltas.

## ✅ Fixed Code

```solidity

function deposit(address token, uint256 amount) external {
    require(isWhitelisted[token], "Token not approved");
    uint256 before = IERC20(token).balanceOf(address(this));
    IERC20(token).transferFrom(msg.sender, address(this), amount);
    uint256 after_ = IERC20(token).balanceOf(address(this));
    require(after_ - before == amount, "Token transfer mismatch");
}
```

## 🛡️ Prevention

### Primary Defenses

- Always check whitelist status (require(isWhitelisted[token])) in every function accepting tokens.
- Validate token behavior using balance deltas, not just return values.
- Reject non-standard or unverified tokens, especially those with delegatecall, mint, or transferFrom overrides.

### Additional Safeguards

- Lock the whitelist with governance voting or timelock before protocol launch.
- Use immutable registries or hash-based token validation (e.g., hashed bytecode).
- Simulate transfers with known tokens in testnet forks and coverage tools.

### Detection Methods

- Static analysis: missing require(isWhitelisted[token]) or unchecked token usage.
- Runtime monitoring of unexpected token addresses or abnormal transferFrom() behavior.
- Tools: Slither (unchecked-token), MythX, Echidna fuzzing for ERC20 edge cases

## 🕰️ Historical Exploits

- **Name:** dYdX Fake Token Listing Attempt 
- **Date:** 2022 
- **Loss:** $9 million 
- **Post-mortem:** [Link to post-mortem](https://dydx.exchange/) 

## 📚 Further Reading

- [SWC-135: Incorrect Token Behavior](https://swcregistry.io/docs/SWC-135) 
- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html) 
- [OpenZeppelin – SafeERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20)

--- 

## ✅ Vulnerability Report

```markdown
id: TBA
title: Token Whitelisting Flaws
severity: H
score:
impact: 4    
exploitability: 4 
reachability: 4  
complexity: 2     
detectability: 3  
finalScore: 3.85
```

---

## 📄 Justifications & Analysis

- **Impact**: Can directly compromise user funds or inflate rewards via fake tokens.
- **Exploitability**: Attacker simply needs to deploy and interact with a malicious ERC20.
- **Reachability**: Common in DeFi vaults, farms, or swaps accepting user-supplied token addresses.
- **Complexity**: Low to medium—deploy fake token or exploit proxy mutation.
- **Detectability**: Detectable via access control, input validation, and token behavior testing.
