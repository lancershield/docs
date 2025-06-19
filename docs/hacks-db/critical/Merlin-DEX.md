# Merlin DEX Insider Exploit 

- **Project**: Merlin DEX
- **Exploit_type**: Privileged Role / Access Control Abuse
- **Loss**: ~$1.8 million (435 WETH + 811 K USDC)
- **Entry_point**: MerlinSwapPair initialization in the DEX pool contracts
- **Exploit_vector**:A privileged “feeTo” address (controller) was granted unlimited token approvals during pool initialization, enabling insiders to drain liquidity once activated.
- **Severity**: Critical
- **Attack_steps**:
    - The MerlinSwapPair contract initialized with approve() granting max token allowances to the feeTo address (controller). 
    - That controller (insider or mismanaged key) had full access to transfer pool tokens.
    - During the Liquidity Generation Event, $1.8M was deposited into the pools. 
    - The controller executed transfers, draining both WETH and USDC. 
    - Funds were migrated out via bridges, some (~$160K) was later frozen by CertiK. 
- **Impact**: Entire liquidity pool drained (≈$1.82M), TVL collapsed, user funds lost.
- **Exploitability**: High – direct access through privileged address.
- **Root_cause**: Critical misuse of privileged initializer (feeTo) granting unlimited approvals—no multisig, no access controls.
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-merlin-dex-hack-april-2023)
