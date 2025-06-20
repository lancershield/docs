# Rari Capital Fuse Exploit 

- **Project**: Rari Capital
- **Exploit_type**: Reentrancy in cToken integration via Fuse Pools
- **Loss**: ~$80 million
- **Entry_point**: exitMarket() function in Compound's forked integration with Fuse
- **Exploit_vector**: Reentrancy during token withdrawal allowed bypassing collateral requirements
- **Severity**: Critical
- **Attack_steps**:
    - Attacker identified a reentrancy issue in how Rari’s Fuse Pools integrated with cTokens.
    - Took out loans using ETH as collateral across multiple Fuse Pools.
    - Called exitMarket() to trigger collateral withdrawal logic.
    - Reentered into the protocol during token transfer via fallback functions.
    - Manipulated logic allowed attacker to withdraw collateral without repaying debt.
    - Repeated the pattern across pools and drained ~$80M worth of assets.
- **Impact**: ~$80 million stolen from Rari’s Fuse lending markets
- **Exploitability**: High
- **Root_cause**: Missing reentrancy guards in Fuse Pool logic interacting with external token contracts
- **Resource**:[Link](https://www.coindesk.com/business/2022/04/30/defi-lender-rari-capitalfei-loses-80m-in-hack)