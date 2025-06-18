# Ronin Network

- **Project**: Ronin Network 
- **Exploit_type**: Private‑Key Compromise + Validator Key Theft
- **Loss**:  $620 million
- **Entry_point**: Bridge validator signatures via submitWithdrawal()
- **Exploit_vector**: Attackers obtained private keys for 5 of 9 validator nodes (4 Sky Mavis + 1 Axie DAO), giving them the authority to forge bridge withdrawals and drain funds.
- **Severity**: Critical
- **Attack_steps**:
    - Gain access to private keys of 4 Sky Mavis validators and 1 Axie DAO validator via compromised RPC or key‑management.
    - Craft forged `submitWithdrawal()` messages with valid signatures from 5 validators.
    - Execute two withdrawal transactions: 173,600 ETH and 25.5 M USDC.
    - Drain funds from the Ronin bridge in a single batch transaction.
    - Exploit went undetected for 6 days until a user withdrawal failed.
    - Funds were later laundered via mixers and traced by Chainalysis teams.
- **Impact**: Entire bridge drained—user ETH and USDC stuck; major integrity and trust collapse .
- **Exploitability**: High – The threshold of 5/9 compromised was sufficient to bypass all multisig controls.
- **Root_cause**:Over‑concentration of validator keys (Sky Mavis held 4 out of 9).Continued use of deprecated gas‑free RPC access for Axie DAO validators.
- **Resource**:[Link](https://www.bbc.com/news/technology-60933174)
