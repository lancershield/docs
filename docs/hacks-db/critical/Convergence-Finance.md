# Convergence Finance Claim Multiple Staking Exploit

- **Project**: Convergence Finance
- **Exploit_type**: Input Validation Flaw in Reward Distribution
- **Loss**: ≈58 million CVG (~$210,000) 
- **Entry_point**: claimMultipleStaking() in 
- **Exploit_vector**: Attacker supplied a malicious contract in the claimContracts array and abused the distributor function to mint 58 M CVG by executing unauthorized logic
- **Severity**: Critical
- **Attack_steps**:
    - Attacker identified the vulnerable claimMultipleStaking() function lacking validation on provided contract addresses.
    - Deployed a malicious contract implementing expected interface (claimCvgCvxMultiple).
    - Included the malicious contract in call to claimMultipleStaking().
    - Function executed malicious logic, minting 58 M CVG.
    - Attacker transferred stolen CVG to mixer addresses and off‑ramped.
- **Impact**: Unauthorized mint of 58 million CVG (~$210K) and draining of reward token reserves
- **Exploitability**: High – input validation flaw easily exploitable
- **Root_cause**: No validation of claimContracts array elements; attacker-supplied contract allowed arbitrary execution
- **Resource**:[Link](https://blog.solidityscan.com/convergence-finance-hack-analysis-12e6acd9ea08) 

