# User-Defined Callback Injection

```YAML
id: TBA
title: User-Defined Callback Injection Enabling Arbitrary Code Execution
severity: H
category: injection
language: solidity
blockchain: [ethereum]
impact: Attacker gains execution control during internal logic flow
status: draft
complexity: high
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-94
swc: SWC-136
```

## 📝 Description

- User-Defined Callback Injection occurs when a smart contract allows a user to specify a callback address or function, which is then called during sensitive logic (e.g., staking, minting, or claiming). If this callback is not properly validated or sandboxed, an attacker can:
- Insert malicious contract calls into the logic flow
- Exploit reentrancy or state confusion
- Hijack control over access-controlled logic or trigger arbitrary code execution.

## 🚨 Vulnerable Code

```solidity
contract CallbackVulnerable {
    mapping(address => uint256) public staked;

    function stake(uint256 amount, address callback) external {
        staked[msg.sender] += amount;

        // ❌ External user-defined callback during internal state flow
        (bool success, ) = callback.call(abi.encodeWithSignature("onStake(address,uint256)", msg.sender, amount));
        require(success, "Callback failed");
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker deploys a contract with a malicious onStake() function that:
2. Reenters stake() or other vulnerable functions,
3. calls back into the same contract to modify sensitive state.
4. Attacker calls stake() on CallbackVulnerable, passing their malicious contract as callback.
5. The call (callback).call(...) executes in the middle of the staking logic, with storage already modified.
6. Reentrancy or privilege abuse leads to drained funds, logic bypass, or state corruption.

**Assumptions:**

- External callback address is controlled by user.
- No access control or isolation applied to the callback execution.
- Callback runs during state transition.

## ✅ Fixed Code

```solidity

contract CallbackSafe {
    mapping(address => uint256) public staked;
    address public approvedCallback;

    modifier onlyCallback() {
        require(msg.sender == approvedCallback, "Not allowed");
        _;
    }

    function setCallback(address _callback) external onlyOwner {
        approvedCallback = _callback;
    }

    function stake(uint256 amount) external {
        staked[msg.sender] += amount;
        // Emit event instead of calling external logic directly
        emit Staked(msg.sender, amount);
    }

    event Staked(address user, uint256 amount);
}
```

## 🛡️ Prevention

### Primary Defenses

- Never allow arbitrary callback addresses to be invoked mid-logic.
- Use event-based signaling instead of external calls (e.g., emit CallbackRequested() instead of .call()).
- If a callback is needed, whitelist it via governance or onlyOwner mechanism.

### Additional Safeguards

- Run external calls after internal state changes (Checks-Effects-Interactions).
- Use reentrancyGuard if callbacks are absolutely necessary.
 Validate that callback does not execute dangerous logic (e.g., delegatecall, transferOwnership, etc.).

### Detection Methods

- Slither: external-call-user-input, callback-injection, and reentrancy detectors.
- Manual inspection of any call(callback) or userFnAddress.delegatecall(...).
- Simulation of edge cases with malicious callbacks using Hardhat or Foundry.

## 🕰️ Historical Exploits

- **Name:** DAO Hack 
- **Date:** 2016 
- **Loss:** ~$60M 
- **Post-mortem:** [Link to post-mortem](https://blog.slock.it/the-dao-hack-explained-62429dbabf62) 


## 📚 Further Reading

- [SWC-136: Unexpected State Changes via Callbacks](https://swcregistry.io/docs/SWC-136) 
- [Consensys – Callback Pattern Dangers](https://consensys.github.io/smart-contract-best-practices/known_attacks/#reentrancy) 
- [Solidity Docs – Low-Level Calls](https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions) 


---
## ✅ Vulnerability Report

```markdown
id: TBA
title: User-Defined Callback Injection Enabling Arbitrary Code Execution
severity: H
score:
impact: 5         
exploitability: 4 
reachability: 4   
complexity: 3     
detectability: 4  
finalScore: 4.3
```


---

## 📄 Justifications & Analysis

- **Impact**: Allows arbitrary control flow hijack, breaking isolation and trust guarantees.
- **Exploitability**: Very straightforward if callback is exposed without checks.
- **Reachability**: Found in bridge protocols, airdrops, staking contracts using dynamic logic hooks.
- **Complexity**: Medium — attacker must deploy and pass malicious contract.
- **Detectability**: Easily found by static analyzers or event tracing through external calls.