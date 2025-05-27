# Blockhash Dependence

```YAML
id: TBA
title: Blockhash Dependence for Randomness 
baseSeverity: H
category: randomness
language: solidity
blockchain: [ethereum]
impact: Predictable or manipulable outcomes due to limited blockhash access
status: draft
complexity: medium
attack_vector: external
mitigation_difficulty: medium
versions: [">=0.4.0", "<0.8.21"]
cwe: CWE-330
swc: SWC-120
```

## 📝 Description

- Blockhash dependence occurs when contracts use `blockhash(block.number - N)` to derive randomness or decision logic. 
- While this may appear secure, `blockhash()` is only reliable for the most recent 256 blocks** and is ultimately manipulable by miners, especially for the current or very recent blocks. 
- This makes it insecure for randomness, lottery draws, or other critical conditions.

## 🚨 Vulnerable Code

```solidity
contract BlockhashLottery {
    function getRandomNumber() public view returns (uint256) {
        return uint256(blockhash(block.number - 1)) % 100; // ❌ insecure randomness
    }

    function drawWinner() external {
        uint256 result = getRandomNumber();
        if (result == 42) {
            // payout logic
        }
    }
}
```

## 🧪 Exploit Scenario

Step-by-step exploit process:

1. Attacker calls drawWinner() using a transaction at the current block.
2. As a miner or via Flashbots, attacker can manipulate blockhash(block.number - 1) by reordering or withholding blocks.
3. They simulate and submit only blocks where result == 42, ensuring they win.
4. This undermines fairness and predictability of the draw.

**Assumptions:**

- The contract relies on recent blockhashes for randomness or security decisions.
- The attacker can simulate and choose when to broadcast transactions or blocks.

## ✅ Fixed Code

```solidity
// Use verifiable randomness via Chainlink VRF
import "@chainlink/contracts/src/v0.8/VRFConsumerBase.sol";

contract SecureLottery is VRFConsumerBase {
    bytes32 internal keyHash;
    uint256 internal fee;
    uint256 public randomResult;

    constructor() VRFConsumerBase(<vrfCoordinator>, <linkToken>) {
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

## 🧭 Contextual Severity

```yaml
- context: "Lottery or game using blockhash for prize logic"
  severity: H
  reasoning: "Predictable randomness allows rigged wins."
- context: "Blockhash used for NFT trait randomization"
  severity: M
  reasoning: "User experience degraded but no financial loss."
- context: "Blockhash used for timestamps or logging only"
  severity: I
  reasoning: "No exploitability—informational only."
```

## 🛡️ Prevention

### Primary Defenses

- Never use blockhash() for randomness or sensitive logic.
- Use verifiable randomness providers like Chainlink VRF, Witnet, or Redstone.

### Additional Safeguards

- If blockhash is used, delay execution across multiple blocks to reduce predictability.
- Use commit-reveal schemes with trusted participants for low-cost alternatives.

### Detection Methods

- Slither: weak-prng, blockhash-dependence detectors.
- Manual inspection of randomness logic relying on blockhash, especially recent ones.
- Symbolic analysis to check miner-controlled inputs.

## 🕰️ Historical Exploits

- **Name:** Roulette Smart Contract Exploit 
- **Date:** 2018 
- **Loss:** Potential manipulation of game outcomes 
- **Post-mortem:** [Link to post-mortem](https://cypherpunks-core.github.io/ethereumbook/09smart-contracts-security.html) 

## 📚 Further Reading

- [SWC-120: Weak Sources of Randomness](https://swcregistry.io/docs/SWC-120) 
- [Solidity Docs – blockhash](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#block-and-transaction-properties) 
- [Chainlink VRF Documentation](https://docs.chain.link/docs/chainlink-vrf/) 

---

## ✅ Vulnerability Report
```markdown
id: TBA
title: Blockhash Dependence for Randomness 
severity: H
score:
impact: 4         
exploitability: 4 
reachability: 4   
complexity: 2     
detectability: 4  
finalScore: 4.0
```

---

## 📄 Justifications & Analysis

- **Impact**: Affects fairness and trust; attacker can rig outcomes in randomness logic.
- **Exploitability**: Easily done if attacker controls block production or uses private mempools.
- **Reachability**: Affects lotteries, NFT traits, game draws, and oracle fallback logic.
- **Complexity**: Requires block manipulation; not high but achievable via miner collusion.
- **Detectability**: Very detectable in code audits and static analysis.
