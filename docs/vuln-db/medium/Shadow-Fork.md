# Shadow Fork Confusion

```YAML
id: TBA
title: Shadow Fork Confusion 
baseSeverity: M
category: environment-confusion
language: solidity
blockchain: [ethereum]
impact: Misconfiguration, accidental value loss, or false security assumptions
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-444
swc: SWC-136
``` 

## 📝 Description

- Shadow forking is a powerful testing feature introduced in tools like Hardhat and Foundry that allows developers to create a local fork of mainnet or other networks for testing with real blockchain state. 
- However, when contracts or accounts are not properly isolated from this simulated environment, developers can:
- Confuse testnet/mainnet addresses
- Replay transactions or leaks into production systems unintentionally
- Mistakenly interact with real mainnet contracts thinking they're mock deployments
- Assume test success or safety that doesn’t hold on actual L1 chains
- This often occurs when deployed addresses (e.g., USDC, WETH, Uniswap) are hardcoded or reused in both testing and production, or if RPC URLs are misconfigured, causing mainnet-state logic to execute during a test or vice versa.

## 🚨 Vulnerable Code

```solidity
pragma solidity ^0.8.0;

interface IUSDC {
    function transfer(address to, uint256 amount) external returns (bool);
}

contract TestSender {
    IUSDC public usdc = IUSDC(0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48); // mainnet USDC

    function send(uint256 amount) external {
        usdc.transfer(msg.sender, amount); // ❌ real mainnet address used in testing
    }
}
```

## 🧪 Exploit Scenario

1. A developer writes tests using mainnet shadow fork to simulate token transfers or DeFi logic.
2. They forget to stub or mock an address like USDC, using the real one from L1.
3. A testing script runs with incorrect RPC or impersonates a funded account via hardhat_impersonateAccount.
4. A state-changing transaction is sent to the real contract, either due to misconfigured RPC or frontend back-end crossover.
5. Real tokens are transferred, or assumptions break due to non-reproducible state divergence.

**Assumptions:**

- Shadow fork environment is misused or misconfigured.
- Code uses mainnet addresses or executes real logic under the assumption of being in a safe sandbox.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

interface IUSDC {
    function transfer(address to, uint256 amount) external returns (bool);
}

contract SafeTestSender {
    IUSDC public usdc;

    constructor(address _usdc) {
        usdc = IUSDC(_usdc); // ✅ injected via constructor
    }

    function send(uint256 amount) external {
        usdc.transfer(msg.sender, amount);
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: M
  reasoning: "Can lead to faulty assumptions or fragile deployments."
- context: "Production contract relying on CREATE2 determinism"
  severity: H
  reasoning: "May brick upgradable paths or lead to loss of control."
- context: "Non-critical simulations or test-only deployments"
  severity: L
  reasoning: "Harmless in isolated testing environments."
```

## 🛡️ Prevention

### Primary Defenses

- Never hardcode L1/L2 addresses in contract logic or test setup.
- Use dependency injection for token, router, or factory addresses.
- Isolate testnet vs production via config and CI/CD environment variables.

### Additional Safeguards

- Use require(block.chainid == <expected>) in critical paths.
- Add RPC safety checks in scripts to prevent real network transactions during testing.
- Monitor gas usage—excessive test costs may indicate wrong network.

### Detection Methods

- Search for known mainnet addresses in test or deploy scripts.
- Check for mainnet RPC usage in .env or CI configs.
- Tools: Slither, grep for 0xA0b8... or common protocol addresses, Foundry's --fork-url

## 🕰️ Historical Exploits

- **Name:** Ethereum Holesky Testnet Shadow Fork Confusion
- **Date:** 2024 
- **Loss:** Potential for testing inconsistencies and misconfigurations due to shadow fork mismanagement 
- **Post-mortem:** [Link to post-mortem](https://www.galaxy.com/insights/research/ethereum-all-core-developers-consensus-call-152/)

## 📚 Further Reading

- [SWC-136: Unobservable Behavior](https://swcregistry.io/docs/SWC-136/) 
- [Hardhat Shadow Forking Guide](https://hardhat.org/hardhat-network/docs/guides/forking-other-networks)
  
## ✅ Vulnerability Report

```markdown
id: TBA
title: Shadow Fork Confusion 
severity: M
score:
impact: 3   
exploitability: 2 
reachability: 4 
complexity: 2    
detectability: 4 
finalScore: 3.0
```

---

## 📄 Justifications & Analysis

- **Impact**: May lead to real funds sent, false security assumptions, or broken logic in production.
- **Exploitability**: Often accidental or internal, but can be manipulated if an attacker has testing access.
- **Reachability**: Frequent in Foundry, Hardhat, or Anvil-based fork testing.
- **Complexity**: Moderate—caused by lack of isolation or proper configuration.
- **Detectability**: Detectable by address audit or RPC configuration checks.