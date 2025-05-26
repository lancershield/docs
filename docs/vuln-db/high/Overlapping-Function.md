# Overlapping Function 
```YAML
id: TBA
title: Overlapping Function  
baseSeverity: H
category: upgradeability
language: solidity
blockchain: [ethereum]
impact: Function hijacking, shadowed logic, or irreversible proxy corruption
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.7.0", "<=0.8.25"]
cwe: CWE-706
swc: SWC-135
```

## 📝 Description

- The Diamond Standard (EIP-2535) enables modular smart contracts where functionality is distributed across multiple facets, with a central diamond proxy routing calls using function selectors. However, if multiple facets define the same function selector (intentionally or accidentally), and selector clashes are not validated during updates, it results in:
- Unintended function routing, where one facet silently overrides another
- Security bypasses, where malicious facets introduce backdoors under existing selectors
- Audit failures, as duplicate logic may appear legitimate but resolve to different implementations
- This is especially dangerous in decentralized or permissionless diamond architectures where updates are proposed via governance or multi-sig, and selector collisions are not resolved or reverted.

## 🚨 Vulnerable Code

```solidity

// Facet A
function transferOwnership(address newOwner) external {
    // legit logic
}

// Facet B (malicious)
function transferOwnership(address newOwner) external {
    selfdestruct(payable(newOwner)); // ❌ same selector, different behavior
}
```

## 🧪 Exploit Scenario

1. A diamond proxy contract manages upgradeable governance, with FacetA defining pause() and transferOwnership().
2. A new facet is added via governance to support a new feature.
3. The facet accidentally (or maliciously) includes a duplicate selector like transferOwnership(), pointing to different logic.
4. The diamond proxy updates its selector table, overwriting the original logic without triggering a revert.
5. The attacker calls transferOwnership() and it now executes malicious logic — such as selfdestruct(), grantRole(), or no-op.

**Assumptions:**

- Diamond implementation does not guard against selector overwrites.
- Facet updates are performed through diamondCut() with insufficient validation.

## ✅ Fixed Code

```solidity

function diamondCut(
    FacetCut[] calldata _facetCuts,
    address _init,
    bytes calldata _calldata
) external {
    for (uint256 i = 0; i < _facetCuts.length; ++i) {
        for (uint256 j = 0; j < _facetCuts[i].functionSelectors.length; ++j) {
            bytes4 selector = _facetCuts[i].functionSelectors[j];
            require(selectorToFacet[selector] == address(0), "Selector collision"); // ✅
            selectorToFacet[selector] = _facetCuts[i].facetAddress;
        }
    }
    // Optional initialization call
}
```

## 🧭 Contextual Severity

```yaml

- context: "Transparent Proxy with selector collision"
  severity: H
  reasoning: "Users or contracts calling logic function may break proxy or trigger admin logic."
- context: "UUPS Proxy with authorization guard"
  severity: L
  reasoning: "Logic isolated; collision risk reduced due to delegation structure."
- context: "Non-proxy contract or externally deployed implementation"
  severity: L
  reasoning: "No impact unless used in proxy setup."
```

## 🛡️ Prevention

### Primary Defenses

- Ensure the diamondCut() function reverts on selector collision unless explicitly overriding.
- Use tooling like DiamondLoupeFacet or Louper to visualize and verify selector state.

### Additional Safeguards

- Add access control to diamondCut to prevent unauthorized updates.
- Generate selectors off-chain and compare across facets before deployment.

### Detection Methods

- Analyze facet code for duplicate function selectors via compiler introspection.
- Tools: 4byte.directory, louper.dev, forge inspect, or hardhat-diamond

## 🕰️ Historical Exploits

- **Name:** NFT Game Diamond Update Error 
- **Date:** 2023-07 
- **Loss:** ~$47,000 
- **Post-mortem:** [Link to post-mortem](https://louper.dev/)

## 📚 Further Reading

- [EIP-2535: Diamond Standard](https://eips.ethereum.org/EIPS/eip-2535) 
- [SWC-135: Incorrect Logic](https://swcregistry.io/docs/SWC-135/) 
- [Diamond Hardhat NPM](https://github.com/mudgen/diamond-3-hardhat) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Overlapping Function 
severity: H
score:
impact: 5    
exploitability: 4 
reachability: 4  
complexity: 3   
detectability: 3 
finalScore: 4.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Function hijacking can lead to asset loss, selfdestruct, or misbehavior.
- **Exploitability**: Facets can be added by governance or multi-sig — sometimes permissionlessly.
- **Reachability**: Applies to all EIP-2535-based proxy systems.
- **Complexity**: Requires internal understanding of selector collisions and facet architecture.
- **Detectability**: Hard to catch during audits without explicit tooling.

