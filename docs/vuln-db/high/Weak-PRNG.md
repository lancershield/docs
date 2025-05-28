# Weak PRNG 

```YAML
id: TBA
title: Weak PRNG  
baseSeverity: H
category: randomness
language: solidity
blockchain: [ethereum]
impact: Manipulated lotteries, unfair game results, or protocol abuse
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-338
swc: SWC-120
```

## 📝 Description

- Solidity does not support secure randomness natively. Developers sometimes attempt to generate random numbers using insecure sources like block.timestamp, blockhash, msg.sender, or tx.origin. 
- These values are publicly known or manipulable, allowing attackers to predict or influence the random output.
- This results in exploitable outcomes for:
- Lotteries
- Token airdrops
- Random NFT traits
- Game results
- Protocol entropy
- Any contract relying on insecure randomness is vulnerable to abuse, especially in front-runnable or miner-influenced environments.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract InsecureRandom {
    function getRandom() public view returns (uint256) {
        return uint256(keccak256(abi.encodePacked(block.timestamp, msg.sender)));
    }
}
```

## 🧪 Exploit Scenario

1. A lottery contract uses block.timestamp + msg.sender as a random seed.
2. Attacker calls getRandom() off-chain until they find a favorable result (e.g., jackpot condition).
3. They then send the transaction with that msg.sender at the right moment.
4. Alternatively, miners can manipulate block.timestamp slightly to influence results.

**Assumptions:**

- Randomness is derived from publicly observable inputs.
- Outcomes are not delayed, verified, or oracle-backed.

## ✅ Fixed Code

```solidity

import "@chainlink/contracts/src/v0.8/VRFConsumerBaseV2.sol";

 contract SecureRandom is VRFConsumerBaseV2 {
    function requestRandomness() external onlyOwner {

    }
    function fulfillRandomWords(...) internal override {
        // use VRF output
    }
 }
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: H
  reasoning: "Randomness predictability results in critical protocol abuse like lottery or gaming manipulation."
- context: "NFT mint randomizer"
  severity: M
  reasoning: "Attackers can target rare trait outcomes, but overall protocol may survive."
- context: "Governance random delegate selection"
  severity: L
  reasoning: "Minor manipulation without direct financial incentive."
```

## 🛡️ Prevention

### Primary Defenses

- Use Chainlink VRF, drand, or other cryptographically secure randomness oracles.
- Delay sensitive operations (e.g., commit-reveal schemes) to break front-running.

### Additional Safeguards

- Combine off-chain commitment with on-chain reveal phases.
- Add entropy from trusted validators or DAOs in coordination.

### Detection Methods

- Search for block.timestamp, blockhash, msg.sender, tx.origin in keccak256().
- Check for randomness used in lotteries, gaming, or mint logic.
- Tools: Slither (weak-prng), MythX, manual audit

## 🕰️ Historical Exploits

- **Name:** Fomo3D Final Round Exploit 
- **Date:** 2018-08-22 
- **Loss:** Over 10,469 ETH (~$3 million) 
- **Post-mortem:** [Link to post-mortem](https://medium.com/rektify-ai/bad-randomness-in-solidity-8b0e4a393858) 
- **Name:** First Flight NFT Puppy Raffle Exploit 
- **Date:** 2023 
- **Loss:** Prize logic manipulated via predictable randomness
- **Post-mortem:** [Link to post-mortem](https://ethereum.stackexchange.com/questions/156027/weak-rng-vulnerability-proving) -->

## 📚 Further Reading

- [SWC-120: Weak PRNG](https://swcregistry.io/docs/SWC-120/) 
- [Chainlink VRF Docs](https://docs.chain.link/vrf) 
- [Preventing the Source of Randomness Vulnerability – Infuy](https://www.infuy.com/blog/preventing-the-source-of-randomness-vulnerability/)
- [How to Safely Generate Random Numbers in Solidity – Medium](https://medium.com/@tiagobertolo/how-to-safely-generate-random-numbers-in-solidity-contracts-bd8bd217ff7b) 
  
--- 

## ✅ Vulnerability Report 

```markdown
id: TBA
title: Weak PRNG Enables Predictable Outcomes in Randomness-Dependent Logic
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 3.8
```

---

## 📄 Justifications & Analysis

- **Impact**: Predictable randomness allows malicious actors to win lotteries, mint rare NFTs, or drain pools unfairly.
- **Exploitability**: Attackers can run local simulations or use timing control (especially miners).
- **Reachability**: Frequently used in on-chain reward/mint logic.
- **Complexity**: Vulnerability is simple to implement and exploit.
- **Detectability**: Easily found via static analysis or manual inspection of hashing logic.
