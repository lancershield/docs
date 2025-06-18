# Cream Finance Exploit 

- **Project**: Cream Finance
- **Exploit_type**: Price Oracle Manipulation
- **Loss**: ~$37.5 million
- **Entry_point**: Lending protocol using Alpha Homora + Iron Bank
- **Exploit_vector**: Exploited a lack of proper validation between Alpha Homora and Cream’s Iron Bank, using recursive borrowing to inflate credit.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker used Alpha Homora v2 to borrow from Iron Bank without actual collateral.
    - They performed recursive calls between the two protocols to inflate credit.
    - Iron Bank assumed Alpha's internal accounting was valid without checking.
    - Attacker extracted ~$37.5M in crypto assets before loan conditions corrected.
- **Impact**: Loss of ETH, USDC, USDT and other tokens via credit delegation flaw.
- **Exploitability**: High
- **Root_cause**: Lack of validation in cross-protocol credit delegation logic between Cream and Alpha.
- **Resource**:[Link](https://www.coindesk.com/business/2021/10/27/cream-finance-exploited-in-flash-loan-attack-worth-over-100m)

