# PancakeBunny Flash Loan Exploit 

- **Project**: PancakeBunny Flash Loan Exploit 
- **Exploit_type**: Flash Loan + Price Manipulation
- **Loss**: ~$45 million
- **Entry_point**: calculatePrice() in the protocol’s reward distribution and price oracle logic
- **Exploit_vector**: The attacker used a flash loan to manipulate the price of BUNNY tokens, then drained the pool by claiming inflated rewards and immediately selling them.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker took out a large flash loan from PancakeSwap.
    - They manipulated the price of BUNNY tokens by swapping assets back and forth in low-liquidity pools.
    - This artificial price increase affected the reward calculation logic.
    - They then deposited collateral to earn inflated BUNNY rewards based on the manipulated price.
    - Immediately sold the freshly minted BUNNY tokens on the open market.
    - Repaid the flash loan and kept the profit from the arbitrage and drained liquidity.
- **Impact**: ~$45M in BUNNY tokens dumped on the market, causing massive price crash and loss of user funds.
- **Exploitability**: High
- **Root_cause**: No protection against price manipulation in the protocol’s reward logic and reliance on on-chain price feeds that could be skewed within a single transaction.
- **Resource**:[Link](https://www.vidma.io/blog/pancakebunny-s-45-million-flash-loan-attack-a-wake-up-call-for-defi-security)