# Velocore DEX Exploit 

- **Project**: Velocore
- **Exploit_type**: Access Control Issue + Fee-Multiplier Logic Flaw
- **Loss**: ~$6.8 million
- **Entry_point**: velocore__execute() in the ConstantProductPool contract
- **Exploit_vector**: Attacker invoked the LP pool action directly—bypassing access control—and manipulated the feeMultiplier, causing underflow in fee calculations to drain the pool.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker directly called velocore__execute() on the LP pool without permission checks. 
    - Supplied crafted parameters to corrupt feeMultiplier, inflating the effective fee beyond 100% and triggering underflow. 
    - Underflow caused a single-token withdrawal to be miscalculated as a deposit, crediting excess LP tokens.
    - Used flash loans to withdraw massive amounts across pools on Linea and zkSync Era. 
    - Moved ~1,800 ETH through Tornado Cash and bridged to other chains. 
    - Linea temporarily halted its sequencer to prevent further outflows. 
- **Impact**: Drained ~$6.8 M from liquidity pools across two chains; TVL collapsed from ~$9.2 M to under $1 M. 
- **Exploitability**: High — no access control and known underflow pattern
- **Root_cause**: Missing authorization in velocore__execute() and failure to enforce safe arithmetic, enabling fee-multiplier underflows
- **Resource**:[Link](https://blog.solidityscan.com/velocore-hack-analysis-642a13630be0)