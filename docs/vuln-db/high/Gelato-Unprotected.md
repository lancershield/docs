# Gelato Unprotected Randomness

```YAML
id: TBA
title: Gelato Unprotected Randomness 
baseSeverity: H
category: randomness
language: solidity
blockchain: [ethereum]
impact: Lottery rigging, reward abuse, or outcome predictability
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-338
swc: SWC-120
```

## 📝 Description

- Gelato is a decentralized automation protocol that offers task execution infrastructure for smart contracts. 
- Some developers use Gelato to schedule or automate functions like lottery draws or airdrops. However, using Gelato’s execution time or transaction context as a randomness source—such as block.timestamp, tx.origin, or block.number—without cryptographic protections introduces deterministic randomness, which can be:
- Predicted off-chain before execution
- Manipulated by bots or miners front-running the Gelato executor
- Exploited to ensure favorable outcomes in lotteries, draws, or reward assignments
- This results in economic manipulation, unfair outcomes, or integrity violations in supposedly random selection processes.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract InsecureRandomGame {
    address public winner;

    function pickWinner(address[] memory players) external {
        require(players.length > 0, "No players");

        // ❌ Predictable randomness based on timestamp
        uint256 rand = uint256(keccak256(abi.encodePacked(block.timestamp, block.number)));
        winner = players[rand % players.length];
    }
}
```

## 🧪 Exploit Scenario

1. A lottery game uses Gelato to automate the pickWinner() function.
2. The randomness is derived from block.timestamp at execution.
3. A bot simulates the function off-chain for upcoming blocks and finds the winning condition.
4. It ensures Gelato runs the task in a block where the outcome is favorable or reorders player addresses in their favor.
5. They win the reward repeatedly or prevent fair distribution.

**Assumptions:**

- Gelato automates execution of functions that rely on block values for randomness.
- No verifiable randomness or commit-reveal is used.
- Attackers can observe or influence block parameters.

## ✅ Fixed Code

```solidity

// Recommended: Use Chainlink VRF or similar verifiable randomness provider

// Example integration
contract SecureRandomGame {
    address public winner;
    bytes32 public keyHash;
    uint64 public subscriptionId;
    VRFCoordinatorV2Interface COORDINATOR;

    constructor(address vrfCoordinator, bytes32 _keyHash, uint64 _subscriptionId) {
        COORDINATOR = VRFCoordinatorV2Interface(vrfCoordinator);
        keyHash = _keyHash;
        subscriptionId = _subscriptionId;
    }

    function requestRandomWinner(address[] memory players) external {
        uint256 requestId = COORDINATOR.requestRandomWords(
            keyHash,
            subscriptionId,
            3,
            200000,
            1
        );
        // Store players in temp storage...
    }

    function fulfillRandomWords(uint256, uint256[] memory randomWords) internal {
        uint256 rand = randomWords[0];
        winner = players[rand % players.length];
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Randomness tied to reward, winner selection, or economic logic"
  severity: H
  reasoning: "Attackers can predict or manipulate outcomes, draining incentives"
- context: "Randomness used for low-value cosmetic effects"
  severity: L
  reasoning: "Impact minimal; only affects UI or animations"
- context: "Randomness sourced from secure oracle or VRF"
  severity: I
  reasoning: "Vulnerability fully mitigated"
```

## 🛡️ Prevention

### Primary Defenses

- Never use block.timestamp, blockhash, tx.origin, or similar for randomness.
- Integrate Chainlink VRF or trusted randomness sources (e.g., RANDAO).

### Additional Safeguards

- Use commit-reveal schemes for user-submitted randomness.
- Validate randomness post-execution using cryptographic proofs.

### Detection Methods

- Search for use of block.timestamp, block.number, or abi.encodePacked(...) used for randomness.
- Tools: Slither (weak-prng), MythX, manual analysis of randomness design

## 🕰️ Historical Exploits

- **Name:** Fomo3D Final Round Exploit 
- **Date:** 2018-08-22 
- **Loss:** ~10,469 ETH 
- **Post-mortem:** [Link to post-mortem](https://medium.com/rektify-ai/bad-randomness-in-solidity-8b0e4a393858)  
- **Name:** First Flight NFT Raffle Exploit 
- **Date:** 2023 
- **Loss:** Raffle manipulated using predictable randomness 
- **Post-mortem:** [Link to post-mortem](https://ethereum.stackexchange.com/questions/156027/weak-rng-vulnerability-proving) 

## 📚 Further Reading

- [SWC-120: Weak Randomness](https://swcregistry.io/docs/SWC-120/)
- [Chainlink VRF Docs](https://docs.chain.link/vrf/v2/introduction)
- [Solidity Docs – Pitfalls of Randomness](https://docs.soliditylang.org/en/latest/security-considerations.html#security-considerations) 
- [Gelato Automation Docs](https://docs.gelato.network/) 

---
  
## ✅ Vulnerability Report

```markdown 
id: TBA
title: Gelato Unprotected Randomness
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.9
```

---

## 📄 Justifications & Analysis

- **Impact**: Predictable randomness undermines protocol fairness and security.
- **Exploitability**: Straightforward to simulate and exploit using public chain data.
- **Reachability**: Common across automated task systems using Gelato or cron-like schedulers.
- **Complexity**: Trivial mistake, yet frequently encountered.
- **Detectability**: Obvious to audits and scanners, though often underestimated.
