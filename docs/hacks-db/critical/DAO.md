# The DAO Hack 

- **Project**: The DAO
- **Exploit_type**: Reentrancy (recursive withdrawal logic flaw)
- **Loss**: ~$60 million 
- **Entry_point**: withdraw() function in The DAO smart contract
- **Exploit_vector**: The attacker repeatedly re-entered the withdraw() function via a fallback loop before the internal balance was updated, draining funds faster than the contract could update state.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker deposited ETH to gain DAO tokens.
    - Called withdraw() to redeem ETH for their DAO tokens.
    - In the token contract’s fallback function (child DAO split), recursively called withdraw() again before original call finalized.
    - Repeated recursion drained ~3.6 M ETH into a child DAO.
    - Attack paused when attacker stopped the recursion—the contract was nearly emptied.
- **Impact**: ~$60 M stolen; subsequently led to Ethereum hard fork and creation of Ethereum Classic 
- **Exploitability**: High – exploit was widely known and trivial once understood
- **Root_cause**: Failure to follow Checks–Effects–Interactions pattern; balance update occurred after external call, enabling reentrancy
- **Resource**:[Link](https://www.gemini.com/cryptopedia/the-dao-hack-makerdao) 
