# TempleDAO Stax Contract Exploit 

- **Project**: TempleDAO
- **Exploit_type**: Missing Access Control on Migration Function
- **Loss**: ~$2.3 million
- **Entry_point**: migrateStake() function in the Stax staking contract
- **Exploit_vector**: The attacker called a publicly accessible migrateStake() function and passed arbitrary user addresses, allowing them to redirect user funds to their own address.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker found that the `migrateStake()` function was publicly callable without any access control.
    - They crafted transactions that impersonated legitimate user migration requests.
    - Instead of sending the staked funds to users’ new wallets, the attacker redirected the migrated stake to their own address.
    - This was repeated across multiple user accounts in a single transaction.
    - Stolen funds were quickly moved out via Tornado Cash and other mixing protocols.
- **Impact**: ~$2.3M in user-staked TEMPLE tokens and assets lost via unauthorized withdrawals.
- **Exploitability**: High
- **Root_cause**: A critical access control oversight—the migrateStake() function lacked validation to restrict who could trigger migrations and to whom funds could be sent.
- **Resource**:[Link](https://www.vidma.io/blog/templedao-hack-a-2-3m-lesson-in-smart-contract-vulnerabilities)

