# Malicious Third-Party Vulnerabilities

```YAML
id: LS42H
title: Malicious Third-Party Vulnerabilities
baseSeverity: H
category: third-party
language: solidity
blockchain: [ethereum, polygon, arbitrum, optimism, bsc]
impact: Unauthorized fund access, logic manipulation, denial of service
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: hard
versions: [">=0.5.0", "<=0.8.25"]
cwe: CWE-829
swc: SWC-107
```

## 📝 Description

- Smart contracts frequently rely on third-party integrations—such as oracles, libraries, DEX routers, plugin hooks, or cross-chain bridges. 
- If these components are untrusted, poorly audited, or mutable, they can introduce critical vulnerabilities into otherwise secure contracts.
- Time-delay bypass or slippage control via compromised oracles
- This issue becomes especially dangerous when the third-party contract is assumed safe but can be upgraded or changed externally.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

interface IPlugin {
    function execute(address target, uint256 amount) external;
}

contract Vault {
    IPlugin public plugin;

    constructor(address _plugin) {
        plugin = IPlugin(_plugin);
    }

    function trigger() external {
        plugin.execute(msg.sender, 1 ether); // ❌ Unverified call to external plugin
    }

    receive() external payable {}
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. A developer integrates a plugin system or a pluggable DEX/router/oracle via an interface without validating the implementation source.
2. The plugin address is either user-configurable or externally controlled (e.g., upgradable proxy owned by a third-party team).
3. The third-party deployer swaps in malicious logic via an upgrade or delegatecall-based mechanism.
4. When trigger() is called in the main contract, it forwards execution to the untrusted plugin, which performs a reentrancy or unauthorized call() draining funds.
5. The vault behaves as if the logic was native, but in reality, execution was hijacked by the plugin, resulting in theft, freezes, or logic corruption.

**Assumptions:**

- Integration depends on externally maintained, upgradeable, or unaudited contract
- No strict interface check or call limitation exists
- The malicious third party intentionally or unintentionally compromises the logic

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeVault {
    address public immutable trustedPlugin;

    constructor(address _plugin) {
        require(_plugin.code.length > 0, "Invalid plugin");
        trustedPlugin = _plugin;
    }

    function trigger() external {
        require(msg.sender == trustedPlugin, "Unauthorized plugin");
        IPlugin(trustedPlugin).execute(msg.sender, 1 ether);
    }

    receive() external payable {}
}
```

## 🧭 Contextual Severity

```yaml
- context: "Protocol accepting arbitrary tokens, NFTs, or oracles"
  severity: H
  reasoning: "Grants execution privileges to potentially hostile contracts"
- context: "Third-party calls wrapped in reentrancy guard and strict allowlist"
  severity: M
  reasoning: "Exposure reduced but still externally influenced"
- context: "Fully internal system or statically linked contracts"
  severity: I
  reasoning: "Vulnerability not applicable"
```

## 🛡️ Prevention

### Primary Defenses

- Audit and freeze third-party contract dependencies
- Use immutable or constant addresses for integrations
- Avoid delegatecall to unknown or user-set addresses

### Additional Safeguards

- Maintain allowlists of authorized integrations
- Require third-party contracts to undergo formal audits or certifications
- Use proxy-safe logic guards when integrating with external upgradables

### Detection Methods

- Manual audit of third-party calls and upgradability mechanisms
- Fuzz tests with swapped plugin contracts to simulate logic change
- Tools: Slither (external-delegatecall, taint-analysis), MythX, Tenderly trace replays

## 🕰️ Historical Exploits

- **Name:** WazirX Multisig Wallet Exploit 
- **Date:** 2024-07 
- **Loss:** $234.9 million 
- **Post-mortem:** [Link to post-mortem](https://medium.com/@somtoochukwu65/smart-contract-hacks-how-hackers-exploit-vulnerabilities-1cde956be814)

## 📚 Further Reading

- [SWC-107: Reentrancy](https://swcregistry.io/docs/SWC-107) 
- [CWE-829: Inclusion of Functionality from Untrusted Control Sphere](https://cwe.mitre.org/data/definitions/829.html) 
- [OpenZeppelin: Avoiding Delegatecall Pitfalls](https://docs.openzeppelin.com/contracts/4.x/api/proxy#transparentproxy)
- [Solidity Docs – External Calls and Reentrancy](https://docs.soliditylang.org/en/latest/security-considerations.html#external-calls) 

## ✅ Vulnerability Report

```markdown
id: LS42H
title: Malicious Third-Party Vulnerabilities
severity: H
score:
impact: 5    
exploitability: 4 
reachability: 3   
complexity: 3    
detectability: 4  
finalScore: 4.15
```

---

## 📄 Justifications & Analysis

- **Impact**: Attacker gains full execution path via delegated or trusted call
- **Exploitability**: Requires plugin upgrade control or pre-positioning
- **Reachability**: Common in protocols using third-party routers, oracles, or plugins
- **Complexity**: Moderate; mostly based on trust assumptions and ownership
- **Detectability**: Auditable with sufficient dependency tracing and fuzzing

