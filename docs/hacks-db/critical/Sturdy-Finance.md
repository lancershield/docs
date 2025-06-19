# Sturdy Finance Oracle Exploit 

- **Project**: Sturdy Finance
- **Exploit_type**: Read-Only Reentrancy + Price Oracle Manipulation
- **Loss**: ~$800,000 (≈ 442 ETH) 
- **Entry_point**: setUserUseReserveAsCollateral() and price-fetch logic using Balancer LP oracle
- **Exploit_vector**: The attacker used a flash loan and Balancer's read-only reentrancy to inflate the price of collateral, then withdrew funds by exploiting the flawed oracle logic during collateral validation.
- **Severity**: Critical
- **Attack_steps**:
    - Took a flash loan of ~110,000 ETH from Aave. 
    - Minted steCRV and B‑stETH‑STABLE tokens via Curve and Balancer pools. 
    - Deposited both tokens as collateral on Sturdy. 
    - Triggered read-only Balancer reentrancy to inflate B‑stETH‑STABLE price nearly 3×.    
    - Called setUserUseReserveAsCollateral() (missing _validateOracle), locking in the inflated price. 
    - Withdrawn steCRV, repaid flash loan partially, and profited the difference. 
- **Impact**: Drained 442 ETH ($800K); several vaults drained; paused all markets. 
- **Exploitability**: High
- **Root_cause**: Lack of reentrancy guard on Balancer oracle and missing oracle validation in collateral toggle function (setUserUseReserveAsCollateral()).
- **Resource**:[Link](https://sturdyfinance.medium.com/exploit-post-mortem-49261493307a)

