# Alpha Homora Exploit

- **Project**: Alpha Homora
- **Exploit_type**: Flash Loan Logic Flaw
- **Loss**: $37.5 million
- **Entry_point**: Iron Bank lending integration with Alpha Homora v2
- **Exploit_vector**: The attacker used a complex chain of flash loans and recursive lending/borrowing between Alpha Homora and Cream’s Iron Bank to manipulate internal accounting and drain funds.
- **Severity**: Critical
- **Attack_steps**: 
    - The attacker initiated a flash loan from dYdX.
    - They recursively borrowed and lent tokens between  Alpha Homora and Iron Bank multiple times.
    - Each iteration tricked the protocol into crediting more collateral than was actually deposited.
    - Once a sufficiently large credit line was created, the attacker withdrew millions in crypto assets.
    - They repaid the flash loan and kept the remaining stolen funds.
  
- **Impact**: USDC, USDT, and other assets were drained from Alpha Homora v2 via Iron Bank credit lines
- **exploitability**: High
- **Root_cause**: Improper validation of cross-protocol credit delegation between Alpha Homora and Iron Bank, allowing recursive loan abuse without real collateral.
- **Resource**:[Link](https://blog.alphaventuredao.io/alpha-homora-v2-post-mortem/)