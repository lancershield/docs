# Qubit Finance hack

- **Project**: Qubit Finance
- **Exploit_type**: Cross‑chain bridge logic bug (Missing input validation)
- **Loss**: ≈ $80 million (206,809 BNB)
- **Entry_point**: deposit() function in QBridge (Ethereum–BSC bridge)
- **Exploit_vector**: Attacker called deposit() with crafted data but no real ETH transfer; contract logic allowed ERC‑20 safeTransferFrom() to empty address, minting unlimited xETH on BSC and enabling mass borrowing 
infosecurity
- **Severity**: Critical
- **Attack_steps**:
    - Called QBridge.deposit() without attaching ETH but with manipulated parameters.
    - safeTransferFrom() to address(0) didn’t revert, passing checks.
    - Ethereum bridge marked deposit valid and triggered minting of xETH on BSC.
    - Used minted xETH as collateral to borrow assets (BNB, tokens).
- **Impact**: 206,809 BNB (~$80 M) drained; seventh‑largest DeFi exploit by value 
- **Exploitability**: High
- **Root_cause**: Deposit function didn’t verify real underlying asset transfer (no code‑existence check on token address); legacy logic left in contract 
halborn.com
- **Resource**:[Link](https://chainalysis.com/blog/qubit-hack-north-korea/)