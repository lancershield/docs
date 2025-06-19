# Dolomite Liquidation Exploit 

- **Project**: Dolomite
- **Exploit_type**: Reentrancy in Legacy Contract via Approval Exploit
- **Loss**: ~$1.8 million in USDC
- **Entry_point**: callFunction() in the deprecated DolomiteMarginProtocol contract
- **Exploit_vector**: The attacker reused old approvals to call callFunction()—bypassing the noEntry guard due to interactions with a secondary contract—enabling unauthorized USDC withdrawals from user wallets.
- **Severity**: Critical
- **Attack_steps**:
    - Targets users who previously granted token approvals to the deprecated protocol (pre-2020). 
    - Attacker triggered callFunction() in the legacy DolomiteMarginProtocol, using the outdated noEntry modifier.
    - Utilized the linked TradeManager contract’s call() method to bypass the reentrancy guard. 
    - Executed arbitrary token transfers, draining USDC from those approved user wallets. 
    - Transferred stolen assets to Tornado Cash and other mixer services. 
    - Dolomite responded by disabling the old contract and advising users to revoke any remaining approvals. 
- **Impact**: ~$1.8 million in USDC stolen from user wallets that retained permissions for deprecated contracts
- **Exploitability**: High
- **Root_cause**: Legacy contract's callFunction() bypassed reentrancy protection via the secondary TradeManager contract—coupled with leftover approved allowances—allowing direct, unauthorized token transfers
- **Resource**:[Link](https://cointelegraph.com/news/old-dolomite-exchange-contract-suffers-1-8-million-loss-from-approval-exploit)
