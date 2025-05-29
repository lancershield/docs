# Sybil Attack Vectors 

```YAML
id: TBA
title: Sybil Attack Vectors 
baseSeverity: H
category: economic-attack
language: solidity
blockchain: [ethereum]
impact: Manipulation of governance, rewards, or fairness
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">0.6.0", "<latest"]
cwe: CWE-302
swc: SWC-114
```
📝 Description

- Sybil attacks occur when a single entity creates multiple accounts (EOAs or contract addresses) to gain disproportionate control or rewards in decentralized systems. These attacks exploit the lack of reliable identity or uniqueness verification in smart contracts.
- Airdrop farming with thousands of wallets
- Voting manipulation in DAOs with pseudo-anonymous accounts
- Multisig approvals with colluding identities
- NFT minting or whitelist abuse by bypassing per-wallet limits
- Since Ethereum allows any actor to generate new EOAs at no cost, systems relying on “one-wallet-one-vote” or “first-come-first-serve” assumptions are inherently vulnerable unless proper Sybil resistance mechanisms are in place.

## 🚨 Vulnerable Code

```solidity

mapping(address => bool) public hasClaimed;

function claimAirdrop() external {
    require(!hasClaimed[msg.sender], "Already claimed");
    hasClaimed[msg.sender] = true;
    token.transfer(msg.sender, 100 * 1e18);
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Protocol launches an airdrop that limits claims to “one per address.”
2. Attacker creates 1,000 EOAs using automated scripts or bot farms.
3. Each EOA calls claimAirdrop() successfully.
4. Attacker consolidates tokens back into a master wallet.
5. Legitimate users are diluted, and attacker gains large, unfair share of protocol funds or governance power.

**Assumptions:**

- No on-chain identity system (e.g., no zkID, POAP, or ENS linkage).
- No anti-Sybil heuristics (e.g., gas usage patterns, clustering).
- Free-to-create EOAs are trusted as distinct users.

## ✅ Fixed Code

```solidity

mapping(bytes32 => bool) public hasClaimed;

function claimAirdrop(bytes32 zkProofRoot) external {
    require(isValidProof(msg.sender, zkProofRoot), "Not eligible");
    require(!hasClaimed[zkProofRoot], "Already claimed");

    hasClaimed[zkProofRoot] = true;
    token.transfer(msg.sender, 100 * 1e18);
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Lack of Sybil resistance enables identity forgery and manipulation."
- context: "Token claim, whitelist, or reward protocol"
  severity: H
  reasoning: "Protocol value directly extracted through mass fake identities."
- context: "Sybil-resistant protocol (e.g., POH, staking)"
  severity: L
  reasoning: "Proper design reduces exploitability through identity proofs or costs."
```

## 🛡️ Prevention

### Primary Defenses

- Use identity-bound proofs (e.g., World ID, Sismo, Gitcoin Passport).
- Link claims to ENS, POAP, or reputational scores.
- Implement gas fee or deposit requirements to discourage mass farming.

### Additional Safeguards

- Delay-based mechanisms: rate-limit claims over time per IP, cluster, or region.
- Batch analysis for Sybil clustering and retroactive filtering.

### Detection Methods

- Static: Flag functions with single-wallet gating (msg.sender) and no identity control.
- Dynamic: Analyze real-time metrics for sybil-like behavior.
- Tools: Forta bots, EigenTrust, SybilRank, Dune dashboards

## 🕰️ Historical Exploits

- **Name:** Gitcoin Passport Sybil Bypass (Mitigated) 
- **Date:** 2022-12 
- **Loss:** Prevented via trust scoring after pattern detection
- **Post-mortem:** [Link to post-mortem](https://passport.gitcoin.co/docs/) 
  
## 📚 Further Reading

- [SWC-133: Hash Collisions With Multiple Inputs](https://swcregistry.io/docs/SWC-133) 
- [Sismo Protocol – Privacy-Preserving Identity](https://docs.sismo.io/) 
- [World ID – Proof of Personhood](https://worldcoin.org/world-id) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Sybil Attack Vectors
severity: H
score:
impact: 3  
exploitability: 4  
reachability: 5  
complexity: 2  
detectability: 3  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Allows attackers to gain disproportionate control or rewards, diluting fairness and undermining trust in reward or governance systems.
- **Exploitability**: Easy to exploit using standard scripts, wallet generators, or Sybil farms; no sophisticated tooling required.
- **Reachability**: Common in DeFi airdrops, NFT mints, DAO voting, and per-wallet reward logic.
- **Complexity**: Requires only basic scripting skills to generate EOAs and automate transactions.
- **Detectability**: Detectable via clustering heuristics, behavioral analytics, or static analysis for lack of identity gating.
