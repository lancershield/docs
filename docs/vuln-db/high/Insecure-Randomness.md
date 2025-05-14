# Insecure Randomness

```YAML

id: TBA
title: Insecure Randomness via Block Data and Predictable Inputs
severity: H
category: randomness
language: solidity
blockchain: [ethereum]
impact: Predictable outcomes in games, lotteries, or rewards
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-330
swc: SWC-120
``` 

## 📝 Description

- Insecure randomness occurs when a contract attempts to generate "random" values using predictable on-chain inputs like `block.timestamp`, `blockhash`, `msg.sender`, or `block.number`. 
- Since miners or attackers can influence or observe these values, the resulting "randomness" can be predicted or manipulated—leading to unfair outcomes in lotteries, games, reward distributions, and token mints.


## 🚨 Vulnerable Code

```solidity
contract Lottery {
    address[] public players;

    function enter() external payable {
        players.push(msg.sender);
    }

    function pickWinner() external {
        uint256 index = uint256(keccak256(abi.encodePacked(block.timestamp, msg.sender))) % players.length;
        payable(players[index]).transfer(address(this).balance);
    }
}
```


## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker observes that pickWinner() uses block.timestamp and msg.sender.
2. They can precompute the winner by simulating the hash locally.
3. If not the winner, they don’t call the function. If yes, they call it and win.
4. In low-activity contracts, attacker may even influence the timestamp (if they're the miner or use front-running).

**Assumptions:**

- No external source of entropy.
- All randomness derived from on-chain, attacker-readable inputs.

## ✅ Fixed Code

```solidity

// Uses Chainlink VRF (Verifiable Random Function)
import "@chainlink/contracts/src/v0.8/VRFConsumerBase.sol";

contract SecureLottery is VRFConsumerBase {
    bytes32 internal keyHash;
    uint256 internal fee;
    uint256 public randomResult;

    constructor()
        VRFConsumerBase(<vrfCoordinator>, <linkToken>)
    {
        keyHash = <your_key_hash>;
        fee = <your_fee>;
    }

    function getRandomNumber() public returns (bytes32 requestId) {
        require(LINK.balanceOf(address(this)) >= fee, "Not enough LINK");
        return requestRandomness(keyHash, fee);
    }

    function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
        randomResult = randomness;
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Use Chainlink VRF, Redstone, Witnet, or other verifiable randomness protocols.
- If off-chain randomness is not an option, use commit-reveal schemes with multiple parties.

### Additional Safeguards

- Use delay or block-lag logic to reduce miner predictability.
- Combine multiple entropy sources including signed off-chain data.

### Detection Methods

- Slither: weak-prng, block-dependent detectors.
- Manual inspection of hash-based randomness using public inputs.
- Static rule engines for entropy misuse.


## 🕰️ Historical Exploits

- **Name:** FairWin Casino Randomness Exploit 
- **Date:** 2019-09-21 
- **Loss:** ~50,000+ ETH drained over time 
- **Post-mortem:** [Link to post-mortem](https://rekt.news/fairwin/) 



- **Name:** Fomo3D Jackpot Snipe 
- **Date:** 2018-07-30 
- **Loss:** N/A (Winner sniped jackpot using block manipulation) 
- **Post-mortem:** [Link to post-mortem](https://www.coindesk.com/markets/2018/07/30/fomo3d-ethereum-gambling-game-sees-investor-win-3-million-prize/)


## 📚 Further Reading

- [SWC-120: Weak Sources of Randomness](https://swcregistry.io/docs/SWC-120) 
- [Chainlink VRF Docs](https://docs.chain.link/vrf) 
- [Trail of Bits – Randomness in Smart Contracts](https://blog.trailofbits.com/2022/07/11/how-to-generate-secure-random-numbers-in-ethereum/) 

---

## ✅ Vulnerability Report 
```markdown
id: TBA
title: Insecure Randomness 
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

- **Impact**: Affected contracts lose trust; attackers can rig games, NFTs, and rewards.
- **Exploitability**: Simple to simulate locally or time with flashbots if predictable.
- **Reachability**: Often exposed through open draw() or pickWinner() logic.
- **Complexity**: Requires understanding of entropy flow, but no special infra.
- **Detectability**: Static analyzers like Slither flag this reliably, but not all hash patterns are caught.