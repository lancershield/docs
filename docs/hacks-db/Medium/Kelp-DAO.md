# Kelp DAO Deposit Slippage Vulnerability

- **Project**: Kelp DAO (rsETH deposit)
- **Exploit_type**: Lack of Slippage Protection → MEV/Front‑Run Risk
- **Loss**:  $10,000 – $50,000 USD
- **Entry_point**: depositAsset() function in the LRTDepositPool contract
- **Exploit_vector**: Without user-specified minimum output (minRSETHOut), attackers could front-run deposits, manipulating price and reducing minted rsETH for users.
- **Severity**: Medium
- **Attack_steps**:
    - User submits deposit that calls depositAsset() without slippage parameter.
    - Attacker monitors pending deposit and front-runs it with a larger deposit.
    - Front-run increases getRsETHPrice() politically.
    - When the original deposit executes, minted rsETH is less than expected.
- **Impact**: Potential user loss via MEV front-running; no actual funds were stolen.
- **Exploitability**: Medium
- **Root_cause**: Omission of slippage control (minRSETHOut) in deposit logic, leaving it vulnerable to front-runners.
- **Resource**:[Link](https://github.com/code-423n4/2023-11-kelp-findings/issues/681)