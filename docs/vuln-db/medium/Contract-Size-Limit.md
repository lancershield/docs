# Contract Size Limit Violations

```YAML
id: TBA
title: Contract Size Limit Violations 
severity: M
category: deployment
language: solidity
blockchain: [ethereum]
impact: Deployment failure or critical logic split across inaccessible contracts
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<0.8.21"]
cwe: CWE-406
swc: SWC-128
```

## 📝 Description

- Contract size limit violations occur when the deployed bytecode of a smart contract exceeds the EVM maximum limit of 24,576 bytes (24 KB). 
- This results in:
- Failed deployments (`exceeds EVM code size limit` error),
- Excessive gas usage and contract bloat,
- Forced use of proxy architectures or unsafe splitting strategies.
- Oversized contracts are difficult to test, maintain, and audit. Additionally, tightly packing logic into a single monolith may hide vulnerabilities across interconnected functions.

## 🚨 Vulnerable Scenario

```solidity
// Multiple inheritance, inline libraries, and bloated storage layouts
contract MassiveContract is A, B, C, D, E {
    // 100+ functions, many structs, mappings, and modifiers
    // Long inline assembly, embedded storage logic
}
```

## 🧪 Exploit Scenario

Step-by-step scenario:

1. Developer adds multiple features (staking, claiming, governance) into one contract.
2. The compiled bytecode exceeds 24 KB.
3. Attempting to deploy results in a hard failure — contract cannot be deployed.
4. As a workaround, they split logic into minimal delegatecalls or unsafe proxies without upgrade management or verification.

**Assumptions:**

- No proxy pattern is used, or the splitting introduces unsafe delegatecall.
- Critical business logic becomes unmanageable or fragmented post-failure.

## ✅ Fixed Approach

``` solidity

// Facet-based design (Diamond pattern or UUPS proxy)
contract CoreLogic {
    function stake() external { /* core logic */ }
}

contract Proxy {
    address public implementation;

    fallback() external payable {
        address impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let success := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch success
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use proxy-based upgradable patterns (UUPS, Transparent, Diamond).
- Split features into separate, importable modules.
- Adopt libraries for reusable logic (e.g., OpenZeppelin’s SafeMath, Ownable, etc.).

### Additional Safeguards

- Run pre-deployment checks (hardhat size-contracts, forge build --sizes).
- Audit code size periodically during development.
- Avoid storing unused logic or feature toggles in production contracts.

### Detection Methods

- Hardhat/Foundry: Use contract-sizer or forge build sizes to monitor bytecode size.
- CI pipelines should reject contracts exceeding safe limits.
- Manual code inspection for bloated inheritance trees or unused inline code.

## 🕰️ Historical 

- **Name:** Uniswap V3 Core 
- **Date:** 2021 
- **Impact:** Code size limitations forced function selector optimizations and minimal proxies 
- **Post-mortem:** [Link to post-mortem](https://uniswap.org/blog/uniswap-v3-core) 


## 📚 Further Reading

- [Solidity Docs – Contract Size Limit](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#contract-size) 
- [OpenZeppelin – Proxy Upgrade Patterns](https://docs.openzeppelin.com/upgrades-plugins/1.x/) 
- [EIP-170: Contract Size Limit](https://eips.ethereum.org/EIPS/eip-170) - [Diamond Standard – Modular Smart Contracts](https://eips.ethereum.org/EIPS/eip-2535) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Contract Size Limit Violations 
severity: M
score:
impact: 3         
exploitability: 2
reachability: 4   
complexity: 2     
detectability: 5  
finalScore: 3.05
```

---

## 📄 Justifications & Analysis

- **Impact**: Deployment blocked or degraded security due to poor workaround (e.g., unverified delegatecall splits).
- **Exploitability**: Not directly exploitable but may introduce new bugs in mitigation (e.g., rushed splitting).
- **Reachability**: Affects all monolithic contract designs or late-stage merged logic.
- **Complexity**: Simple oversight—hard to fix post-merge.
- **Detectability**: Always visible via compiler or deployment error—build tooling must enforce.
