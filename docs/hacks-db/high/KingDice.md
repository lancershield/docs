# KingDice Predictable RNG Exploit

- **Project**: KingDice
- **Exploit_type**: Predictable Random Number Generator (RNG)
- **Loss**: ~$18,000
- **Entry_point**: roll() function in the betting smart contract
- **Exploit_vector**: Attacker reverse-engineered the RNG logic based on block variables to always predict winning outcomes
- **Severity**: High
- **Attack_steps**:
    - Analyzed smart contract’s RNG logic
    - Identified reliance on predictable block attributes (e.g., block.timestamp, block.number)
    - Used this to predict winning dice outcomes
    - Placed repeated bets and drained funds
- **Impact**: Over 210 ETH (~$18K at the time) was siphoned by exploiting predictable randomness
- **Exploitability**: High — no cryptographic randomness or oracle used
- **Root_cause**: Use of on-chain, non-secure randomness that could be predicted by miners or attackers
- **Resource**:[Link](https://stackoverflow.com/questions/23147385/how-to-exploit-a-vulnerable-prng)