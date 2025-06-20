# Bogged Finance Price Manipulation Exploit 

- **Project**: Bogged Finance
- **Exploit_type**: Flash Loan + Price Oracle Manipulation
- **Loss**: ~$3 million
- **Entry_point**: BogToken smart contract relying on BNB/BOG price pair from PancakeSwap
- **Exploit_vector**: Attacker manipulated BOG/BNB liquidity pool to inflate token value before selling
- **Severity**: High
- **Attack_steps**:
    - Attacker took a flash loan of BNB to gain instant capital.
    - Used it to heavily manipulate the BOG/BNB trading pair on PancakeSwap, spiking the price of BOG.
    - Exploited the manipulated price to mint more BOG tokens or trigger internal mechanics favoring inflated value.
    - Swapped BOG tokens at the inflated rate to drain paired assets (BNB).
    - Repaid the flash loan and retained ~$3M worth of BNB from the manipulated trade.
    - The liquidity pool was left imbalanced, and BOG price collapsed.
- **Impact**: ~$3 million worth of BNB drained from the BOG/BNB liquidity pool
- **Exploitability**: High
- **Root_cause**: Insecure reliance on real-time DEX pricing without anti-manipulation checks or TWAP logic
- **Resource**:[Link](https://chainbulletin.com/bogged-finance-suffers-3m-flash-loan-exploit)
