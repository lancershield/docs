# Parity Multisig Wallet Hack

- **Project**: Parity Multisig Wallet
- **Exploit_type**: Access Control / Delegatecall Initialization Bug
- **Loss**: ~153,037 ETH (~US $30 million) 
- **Entry_point**: initWallet() function in the shared wallet library contract (used via delegatecall) 
- **Exploit_vector**: Attackers called initWallet() via fallback, resetting owners to themselves; then executed fund transfers. Months later, another attacker initialized and self-destructed the library, freezing funds 
- **Severity**: Critical 
- **Attack_steps**:
    - Attacker identified unprotected initWallet() in library accessible via delegatecall.
    - Sent transaction to execute initWallet(), setting themselves as sole owner and reducing required confirmations to 1.
    - Called execute() to drain funds from multisig wallets.
    - In November, attacker invoked initWallet() on the library itself then called selfdestruct(), deleting the library and freezing linked wallets 
- **Impact**: ~153k ETH stolen; ~514k ETH irreversibly frozen; tens/hundreds of wallets affected, including ICO and foundation funds 
- **Exploitability**: High 
- **Root_cause**: Poor visibility and guard design in library usage: public initWallet() without access control; misuse of delegatecall; shared uninitialized library with selfdestruct() left executable by anyone 
- **Resource**:[Link](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7)

