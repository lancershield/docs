# El Dorado Exchange Oracle Manipulation Exploit 

- **Project**: El Dorado Exchange (EDE Finance)
- **Exploit_type**: Price Oracle Manipulation
- **Loss**: ~$520,000 (437,948 USDC + 86,222 USDT) 
- **Entry_point**: Closed-source Oracle contract via low-level function (func_147d9322)
- **Exploit_vector**: Attacker manipulated on-chain oracle by exploiting a vulnerable function to feed false token prices and drain liquidity through arbitrage positions.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker identified and invoked a hidden oracle update function (func_147d9322) within the closed-source Oracle contract.
    - Used this to feed artificially low token prices into the system.
    - Executed arbitrage trades based on the manipulated price data across EDE pools.
    - Swapped tokens to extract value at distorted prices.
- **Impact**: Drained ~437,948 USDC and 86,222 USDT ($520K) from the protocol.
- **Exploitability**: High — oracle misconfiguration allowed unauthorized price updates
- **Root_cause**: Improper access control: critical oracle update function was exposed and callable without restrictions.
- **Resource**:[Link](https://faun.pub/blockchain-weekly-379-a-detailed-analysis-on-ede-finances-520k-hack-4e45e37947d6)