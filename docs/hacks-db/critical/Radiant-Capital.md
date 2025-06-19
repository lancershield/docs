# Radiant Capital Exploit 

- **Project**: Radiant Capital
- **Exploit_type**: Flash Loan + Rounding Error (Precision Calculation Bug)
- **Loss**: ~$4.5 million 
- **Entry_point**: rayDiv() arithmetic-and-state function during new market initialization + flash loan logic
- **Exploit_vector**: Attacker manipulated small pool initialization and a liquidity index calculation with a rounding error, enabling them to withdraw more funds than they deposited via flash loan cycles.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker took a ~$3M USDC flash loan from Aave. 
    - Deposited 2M USDC into a newly deployed Radiant USDC market with uninitialized totalSupply. 
    - In flash loan callback, liquidated liquidity index by swapping withdrawals, triggering rounding discrepancies in rayDiv(). 
    - Repeated deposit-withdraw loops (~18 iterations), draining ~$2.8M via the liquidity index error. 
    - Repaid flash loan and extracted ~1902 ETH. 
- **Impact**: ~$4.5M stolen; damaged cross-chain lending markets; protocol paused for emergency refix. 
- **Exploitability**: High
- **Root_cause**: Rounding/precision flaw in rayDiv() calculation when handling initial low-liquidity markets; lack of safe-initialization and parameter validation.
- **Resource**:[Link](https://blog.solidityscan.com/radiant-capital-hack-analysis-b300ebdeee29?gi=d8567282a14f)

