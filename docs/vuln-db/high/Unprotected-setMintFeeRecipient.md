# Unprotected setMintFeeRecipient

```YAML
id: LS52H
title: Unprotected setMintFeeRecipient 
baseSeverity: H
category: access-control
language: solidity
blockchain: [ethereum]
impact: Unauthorized fee redirection
status: draft
complexity: low
attack_vector: external
mitigation_difficulty: easy
versions: [">=0.6.0"]
cwe: CWE-284
swc: SWC-105
```

## 📝 Description

- In token minting contracts where users pay a fee per mint, the contract typically includes a function like setMintFeeRecipient() to control where the fees are sent. If this function:
- Lacks proper access control (e.g., onlyOwner)
- Can be called publicly or by any address
- Can be overwritten by governance exploits or fallback proxies
- Then any external actor could change the fee recipient, allowing them to drain all mint proceeds without owning the minting logic itself.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract NFTMinter {
    address public mintFeeRecipient;
    uint256 public mintFee = 0.01 ether;

    function setMintFeeRecipient(address newRecipient) external {
        mintFeeRecipient = newRecipient; // ❌ no access control
    }

    function mint() external payable {
        require(msg.value >= mintFee, "Insufficient fee");
        payable(mintFeeRecipient).transfer(mintFee);
        // Mint logic omitted
    }
}
```

## 🧪 Exploit Scenario

1. Protocol deploys a public mint with an unguarded setMintFeeRecipient() function.
2. Attacker notices no onlyOwner modifier and calls

**Assumptions:**

- Contract has active users calling mint()
- No proxy delay, governance, or admin system secures recipient logic

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureNFTMinter is Ownable {
    address public mintFeeRecipient;
    uint256 public mintFee = 0.01 ether;

    function setMintFeeRecipient(address newRecipient) external onlyOwner {
        require(newRecipient != address(0), "Zero address");
        mintFeeRecipient = newRecipient;
    }

    function mint() external payable {
        require(msg.value >= mintFee, "Insufficient fee");
        payable(mintFeeRecipient).transfer(mintFee);
        // Safe mint logic
    }
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: H
  reasoning: "Allows external redirection of core protocol revenue."
- context: "Protocol that collects high minting fees"
  severity: C
  reasoning: "Massive financial impact from misrouted or stolen fees."
- context: "Internal test contract"
  severity: I
  reasoning: "No meaningful risk when used in controlled environments."
```

## 🛡️ Prevention

### Primary Defenses

- Always guard admin/setter functions with access control (Ownable, AccessControl)
- Validate new values (e.g., not zero, not current caller)

### Additional Safeguards

- Emit FeeRecipientUpdated event for transparency
- Use timelocks or multisig for high-value minting contracts

### Detection Methods

- Search for setter functions with public visibility and no access control
- Tools: Slither (unprotected-upgradeable, missing-authorization), MythX, manual review

## 🕰️ Historical Exploits

- **Name:** Generic NFT Launchpad Config Bug 
- **Date:** 2021-10 
- **Loss:** N/A (found during audit) 
- **Post-mortem:** [Link to post-mortem](https://blog.openzeppelin.com/) 
  
## 📚 Further Reading

- [SWC-105: Unprotected Critical Function](https://swcregistry.io/docs/SWC-105/) 
- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html) 
- [OpenZeppelin Ownable Contract](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable) 

---

## ✅ Vulnerability Report

```markdown
id: LS52H
title: Unprotected setMintFeeRecipient
severity: H
score:
impact: 5   
exploitability: 4 
reachability: 4  
complexity: 2     
detectability: 5  
finalScore: 4.25
```

---

## 📄 Justifications & Analysis

- **Impact**: Protocol loses all minting revenue; users may be unknowingly funding an attacker
- **Exploitability**: Anyone can hijack the fee flow with a single transaction
- **Reachability**: Present in many minimal or rapid-launch NFT/token mints
- **Complexity**: Simple implementation flaw; no special knowledge needed
- **Detectability**: Trivial to catch with Slither or manual inspection