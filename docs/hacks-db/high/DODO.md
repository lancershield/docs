# DODO

- **Project**: DODO
- **Exploit_type**: Insecure Contract Initialization
- **Loss**: ~$3.8 million
- **Entry_point**: _init() function in newly deployed DODO V2 Crowdpooling smart contracts
- **Exploit_vector**: Attacker initialized unprotected contracts with their own logic before DODO could
- **Severity**: High
- **Attack_steps**:
    - DODO used a factory contract to deploy new crowdpooling contracts using create2.
    - These contracts were meant to be initialized only once using the _init() function.
    - However, _init() lacked access control and was publicly callable.
    - The attacker monitored for new contract deployments before DODO initialized them.
    - The attacker quickly called _init() themselves and set malicious token addresses.
    - After hijacking the contracts, they drained the funds held by these contracts.
- **Impact**: Several pools lost funds including $WETH, $USDT, and $USDC; some recovered, others permanently lost
- **Exploitability**: Medium
- **Root_cause**: No access restriction on the _init() function in newly deployed contracts
- **Resource**:[Link](https://decrypt.co/60712/defi-protocol-dodo-hacked-for-3-8-million)