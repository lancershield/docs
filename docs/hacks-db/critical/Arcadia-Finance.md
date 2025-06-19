# Arcadia Finance Reentrancy Exploit 

- **Project**: Arcadia Finance
- **Exploit_type**: Incomplete Reentrancy Protection in Liquidation Logic
- **Loss**: ~$455,000 
- **Entry_point**: liquidateVault() via vaultManagementAction() in the Vault contract
- **Exploit_vector**: The attacker used a malicious contract to re-enter the liquidation logic mid-execution, bypassing health checks and protocol integrity, effectively draining vault collateral repeatedly.
- **Severity**: Critical
- **Attack_steps**:
    - Took a flash loan (2.4 WETH + ~20K USDC) from Aave on Ethereum and Optimism 
    - Deposited borrowed funds into a leveraged vault using doActionWithLeverage()
    - Within vault logic, supplied a malicious actionHandler contract to vaultManagementAction()
    - Malicious handler re-entered liquidateVault() before collateral health check completed 
    - This bypass allowed funds withdrawal while vault state was still "healthy," nullifying checks
    - Repeated the sequence across Ethereum and Optimism vaults—draining ~$455K total 
- **Impact**: Protocol funds drained across chains; TVL dropped ~77%, user positions frozen. 
decrypt.co
- **Exploitability**: High – reentrancy flaw is well-known and exploitable in low-complexity scenarios
- **Root_cause**: Lack of reentrancy guard and improper state validation ordering in liquidation logic, allowing mid-transaction health checks to be bypassed
- **Resource**:[Link](https://immunebytes.com/blog/arcadia-finance-exploit-detailed-hack-analysis/)