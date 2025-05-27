# Front-Running NFT Minting

```YAML
id: TBA
title: Front-Running NFT Minting 
baseSeverity: H
category: front-running
language: solidity
blockchain: [ethereum]
impact: Attackers can steal rare mints or outpace users for whitelist slots
status: draft
complexity: medium
attack_vector: mempool
mitigation_difficulty: medium
versions: [">=0.6.0", "<latest"]
cwe: CWE-362
swc: SWC-114
```

## 📝 Description

- Front-running NFT minting occurs when a malicious actor observes pending mint transactions in the public mempool and submits a competing transaction with higher gas, thereby:
- Sniping rare NFTs intended for another user,
- Minting before supply caps are hit,
- Bypassing whitelist or reserved logic that isn't enforced on-chain.
- This exploit is especially common in:
- Public NFT mints with randomized token IDs,
- First-come-first-served presales,
- Blind mints where token assignment is deterministic or guessable.

## 🚨 Vulnerable Code

```solidity
function mint(uint256 amount) public payable {
    require(totalMinted + amount <= MAX_SUPPLY, "Sold out");
    require(msg.value >= amount * price, "Insufficient ETH");

    for (uint256 i = 0; i < amount; i++) {
        _mint(msg.sender, totalMinted + i); // ❌ Deterministic tokenId
        totalMinted++;
    }
}
```

## 🧪 Exploit Scenario

Step-by-step attack:

1. A user broadcasts a transaction to mint 1 NFT via mint(1) with expected token ID 1001.
2. An attacker observes the pending transaction and submits their own mint(1) with higher gas.
3. The attacker’s transaction gets mined first and receives token ID 1001.
4. The user’s transaction now mints token ID 1002 — they’ve lost the rare or expected token.

**Assumptions:**

- Attacker submits a batch mint to exhaust supply right before others.
- Randomization or whitelisting logic is not enforced in mint() function, only off-chain.

## ✅ Fixed Code

```solidity

// ✅ Use commit-reveal or VRF for token assignment
function mint() external {
    require(mintingOpen, "Minting not active");
    uint256 tokenId = getRandomTokenId(); // e.g. Chainlink VRF
    _mint(msg.sender, tokenId);
}

// ✅ Alternatively: Pre-commit and reveal
mapping(address => bytes32) public commitments;

function commit(bytes32 hash) external {
    commitments[msg.sender] = hash;
}

function reveal(uint256 salt, uint256 amount) external {
    require(keccak256(abi.encodePacked(salt, amount)) == commitments[msg.sender], "Invalid reveal");
    // proceed to mint
}
```

## 🧭 Contextual Severity

```yaml
- context: "Public mint with known tokenId sequencing and valuable traits"
  severity: H
  reasoning: "High-value tokens can be sniped by attackers"
- context: "Minting is randomized or uses commit-reveal"
  severity: M
  reasoning: "Attack surface reduced significantly"
- context: "Whitelist or gating prevents front-running"
  severity: L
  reasoning: "Minimal or no exposure to public mempool race"
```

## 🛡️ Prevention

### Primary Defenses

- Use commit-reveal schemes or Chainlink VRF/randomness to prevent predictable token assignment.
- Implement on-chain whitelists or signature verification (EIP-712).
- Add per-wallet rate limits or caps per block.

### Additional Safeguards

- Allow minting only in private batches or allowlist time windows.
- Allow minting via backend signature authorization, requiring EOA validation.
- Disable direct minting in public contracts and route through a controlled interface.

### Detection Methods

- Slither: predictable-tokenid, public-mint-without-randomness, missing-whitelist-check.
- Simulation of gas-race scenarios on testnets with multiple actors.
- Mempool analysis of gas bidding behavior during NFT drops.

## 🕰️ Historical Exploits

- **Name:** Magic Eden Ordibots Front-Running Incident 
- **Date:** November 2023 
- **Loss:** Unspecified financial losses due to failed NFT purchases 
- **Post-mortem:** [Link to post-mortem](https://protos.com/how-a-quant-sniped-millions-from-bitcoin-ordinals/) 

## 📚 Further Reading

- [SWC-114: Unrestricted Actions](https://swcregistry.io/docs/SWC-114) 
- [Chainlink VRF Docs](https://docs.chain.link/vrf) 
- [Slither Front-Running Detectors](https://github.com/crytic/slither)

---

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Front-Running NFT Minting 
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 3   
complexity: 3     
detectability: 4  
finalScore: 3.85
```

---

## 📄 Justifications & Analysis

- **Impact**: High — destroys fairness of minting and affects community trust.
- **Exploitability**: High — front-running bots can execute easily.
- **Reachability**: Common in NFT drop contracts lacking randomness.
- **Complexity**: Moderate — but bots and mempool watchers make it easy.
- **Detectability**: High — deterministic logic is visible in audit or test.