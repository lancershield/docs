# Eminence Finance Reentrancy Token Mint Exploit

- **Project**: Eminence Finance (EMN)
- **Exploit_type**: Reentrancy / Lack of Access Control on Mint Function
- **Loss**: ~$15 million 
- **Entry_point**: public mint() and burn() functions in the EMN smart contract
- **Exploit_vector**: Attacker invoked mint() repeatedly via reentrancy without proper guard, creating unlimited EMN tokens—then issued trades and dumped tokens for native assets
- **Severity**: Critical
- **Attack_steps**:
    - Deployed and published EMN contract to mainnet without access controls.
    - Invoked mint() to create EMN tokens arbitrarily.
    - Re-entered mint() through recursive calls to inflate balance exponentially.
    - Used burn() and swap functions to convert minted EMN into ETH and stablecoins.
    - Repeated until ~$15M worth of value was extracted. 
- **Impact**: ~$15M stolen; attacker later returned ~$8M to Yearn deployer address—source of recovery is the deployer’s multisig
- **Exploitability**: High — unrestricted mint functions and no reentrancy protection made it trivial
- **Root_cause**: Critical oversight in allowing public mint/burn with no access control or reentrancy guard
- **Resource**:[Link](https://decrypt.co/43203/hackers-drain-15-million-from-unreleased-yearn-finance-project) 
