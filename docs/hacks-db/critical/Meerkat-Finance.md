# Meerkat Finance Rug Pull 

- **Project**: Meerkat Finance
- **Exploit_type**: Deployer Contract Tampering (Insider Rug Pull)
- **Loss**: ~$31 million
- **Entry_point**: Vault contract controlled by the deployer
- **Exploit_vector**: The deployer changed the vault’s logic to redirect user funds to their own address shortly after launch, effectively draining the protocol under the guise of a “hack.”
- **Severity**: Critical
- **Attack_steps**:
    - Meerkat Finance launched as a yield farming platform on BSC, promising high APRs.
    - Within 24 hours of going live, the deployer modified the smart contract logic using upgradeable proxies.
    - The modified contract redirected user funds to external wallets controlled by the attacker.
    - Around $13 million in BUSD and $18 million in BNB were withdrawn from user vaults.
    - The project deleted its social media and website shortly after the event.
    - Initially framed as an external exploit, but on-chain analysis confirmed insider activity.
- **Impact**: ~$31M drained from user vaults—funds permanently lost, and community abandoned.
- **Exploitability**: High
- **Root_cause**: Complete control retained by the deployer over upgradeable contracts, allowing arbitrary code replacement with no timelock, multisig, or transparency.
- **Resource**:[Link](https://www.vidma.io/blog/meerkat-finance-anatomy-of-a-31-million-defi-rug-pull-on-binance-smart-chain) 