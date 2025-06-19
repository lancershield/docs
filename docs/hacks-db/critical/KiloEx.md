# KiloEx Exchange Oracle Exploit 

- **Project**: KiloEx (Decentralized Exchange, Base, BNB, Taiko)
- **Exploit_type**: Price Oracle Manipulation via Access-Control Flaw
- **Loss**: ~$7.5 million 
- **Entry_point**: setPrices() in KiloPriceFeed via MinimalForwarder contract
- **Exploit_vector**: Attacker exploited a permissionless MinimalForwarder to manipulate on-chain oracle, then opened and closed leveraged positions at artificially skewed prices
- **Severity**: Critical
- **Attack_steps**:
    - Attacker invoked execute() on MinimalForwarder, supplying forged caller data. 
    - This call chained to PositionKeeper → Keeper → KiloPriceFeed, allowing setPrices() execution. 
    halborn.com
    - Oracle price was set drastically low (e.g. ETH/USD at 100). 
    - Attacker opened large leveraged positions at deflated price.
    - Next, artificially inflated the oracle price (e.g. ETH/USD to 10,000). 
    - Closed positions at inflated price, extracting excessive profits.
    - Repeated across Base, BNB Chain, and Taiko networks, draining ~$7.5M. 
- **Impact**: ~$7.5M drained from leveraged positions; cross-chain liquidity halted and positions refunded
- **Exploitability**: High – exploit leveraged existing access control flaw in a critical oracle path
- **Root_cause**: Insecure MinimalForwarder.execute() logic allowed unauthorized upstream oracle access via indirect permission chain
- **Resource**:[Link](https://cointelegraph.com/news/kiloex-exchange-exploiter-returns-5-5-m-7-5-m-dex-exploit 
onesafe.io)
