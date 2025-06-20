# Synthetix Oracle Exploit

- **Project**: Synthetix
- **Exploit_type**: Oracle Manipulation / Faulty Price Aggregation
- **Loss**: 37 million 
- **Entry_point**: FX Oracle for Korean Won (sKRW) pricing
- **Exploit_vector**: A bot exploited an incorrect price from the centralized oracle by rapidly swapping sKRW (valued at 1000x its real price) for sETH
- **Severity**: Critical
- **Attack_steps**:
    - Two out of three external KRW price feeds failed.
    - The remaining feed reported a price ~1000x higher.
    - Oracle averaged the feeds without proper outlier rejection, causing inflated price.
    - An arbitrage bot detected this and swapped sKRW to sETH repeatedly.
    - Over 37 million sETH were minted before the team paused the system.
    - The team negotiated with the bot operator and recovered the assets through a bug bounty deal.
- **Impact**: Massive synthetic asset imbalance; trading halted temporarily; potential $1B exploit avoided through coordination
- **Exploitability**: High – oracle had no fallback or anomaly detection logic
- **Root_cause**: Failure to implement robust oracle aggregation logic and lack of redundancy; trusted centralized price feeds without validating correctness
- **Resource**:[Link](https://messari.io/report/synthetix-suffers-an-oracle-attack-that-lost-roughly-37-million)