# Arbitrary from in transferFrom

```YAML
id: TBA
title: Arbitrary from in transferFrom
baseSeverity: C
category: authorization-bypass
language: solidity
blockchain: [ethereum, polygon, bsc, arbitrum, optimism]
impact: Unauthorized token transfer from arbitrary user accounts
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-285
swc: SWC-114
```

## 📝 Description

- The transferFrom() function in ERC-20 or ERC-721 tokens is designed to transfer tokens on behalf of another address, but it must validate that the caller is authorized to do so (i.e., msg.sender has a sufficient allowance from from).
- If a custom token implementation does not enforce proper access control on the from parameter, it allows any user to drain tokens from any address, resulting in theft of assets. 
- This bug typically occurs when the contract checks only balances[from] without validating allowance[from][msg.sender].

## 🚨 Vulnerable Code

```solidity

function transferFrom(address from, address to, uint256 amount) public returns (bool) {
    require(balances[from] >= amount, "Insufficient balance");
    balances[from] -= amount;
    balances[to] += amount;
    return true; // ❌ missing allowance check
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A custom ERC-20 token contract is deployed with the above transferFrom() implementation.
2. Alice holds 1,000 tokens but has not approved anyone to spend her tokens.
3. Attacker calls transferFrom(alice, attacker, 1_000) directly.
4. The contract checks Alice’s balance but not whether the attacker is allowed to use it.
5. Tokens are transferred from Alice to the attacker without consent.

**Assumptions:**

- The token uses a broken transferFrom() implementation.
- msg.sender is not validated as being approved by from.
- Victims have token balances in the vulnerable contract.

## ✅ Fixed Code

```solidity

function transferFrom(address from, address to, uint256 amount) public returns (bool) {
    require(balances[from] >= amount, "Insufficient balance");
    require(allowances[from][msg.sender] >= amount, "Not approved");

    allowances[from][msg.sender] -= amount;
    balances[from] -= amount;
    balances[to] += amount;
    return true;
}
```

## 🧭 Contextual Severity

```yaml
- context: "Core ERC-20 token logic lacks allowance checks"
  severity: C
  reasoning: "Anyone can drain any user’s balance—catastrophic financial loss."
- context: "ERC-20 wrapper or internal logic used by known dApps"
  severity: H
  reasoning: "Still very dangerous; attacker may exploit via integrations."
- context: "transferFrom() unused, or only called internally with trusted addresses"
  severity: M
  reasoning: "Risk depends on usage context; still a dangerous pattern to leave in code."
```

## 🛡️ Prevention

### Primary Defenses

- Always validate that msg.sender is approved via allowance[from][msg.sender].
- Use OpenZeppelin’s ERC-20 or ERC-721 contracts.
- Never reinvent token logic unless necessary and reviewed.

### Additional Safeguards

- Add tests that simulate transfers from unauthorized addresses.
- Include fail-safe ownership guards on custom transfer logic.
- Audit external calls that pass user addresses as parameters.

### Detection Methods

- Slither: missing-authorization, custom-erc20-transferfrom
- Manual review of transferFrom() implementations
- Static assertions: check presence of allowance validation

## 🕰️ Historical Exploits

- **Name:** Visor Finance vVISR Staking Contract Exploit
- **Date:** 2021-12-21
- **Loss:** Approximately 8.8 million VISR tokens
- **Post-mortem:** [Link to post-mortem](https://medium.com/visorfinance/post-mortem-for-vvisr-staking-contract-exploit-and-upcoming-migration-7920e1dee55a)

## 📚 Further Reading

- [SWC-114: Authorization Bypass](https://swcregistry.io/docs/SWC-114)
- [CWE-285: Improper Authorization](https://cwe.mitre.org/data/definitions/285.html)
- [OpenZeppelin ERC20 Reference](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20)
- [Solidity Docs – Function Modifiers](https://docs.soliditylang.org/en/latest/contracts.html#function-modifiers) 
  
---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Arbitrary from in transferFrom
severity: C
score:
impact: 5
exploitability: 5
reachability: 5
complexity: 1
detectability: 4
finalScore: 4.75
```

---

## 📄 Justifications & Analysis

- **Impact**: Results in complete loss of user funds due to unauthorized token transfers.
- **Exploitability**: Easily exploitable by anyone calling the public function.
- **Reachability**: High—transferFrom() is designed to be public and widely used.
- **Complexity**: Low—attacker simply calls the function with arbitrary from.
- **Detectability**: Easily detectable by audit tools or during review, but frequently overlooked in custom token implementations.
