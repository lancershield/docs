# Multichain Router Permit Vulnerability Exploit 

- **Project**: Multichain 
- **Exploit_type**: permit() Misuse + Input Validation Flaw
- **Loss**: ~$3 million stolen from users
- **Entry_point**: anySwapOutUnderlyingWithPermit() in the Multichain router contract
- **Exploit_vector**: Attackers abused the router’s permit-based withdrawal function to transfer users’ approved tokens—without checking if the target token was a legitimate “anyToken”—and drained funds from wallets that had given approval.
- **Severity**: Critical
- **Attack_steps**:
    - Vulnerability existed in January 2022 when Multichain added anySwapOutUnderlyingWithPermit() functionality.
    - Function used permit() to auto-approve token transfers during cross-chain swaps.
    - It failed to validate whether the token parameter was a genuine Multichain-wrapped token. 
    - Attackers exploited this flaw by calling the function with user-approved underlying tokens (e.g., WETH) directly—bypassing checks.
    - Funds were pulled from users’ wallets into attacker-controlled addresses using legitimate-looking calls.
    - Approximately $3 million across six token types were drained before patching. 
- **Impact**: ~$3 M in user funds stolen across WETH, WBNB, MATIC, AVAX, PERI, OMT
- **Exploitability**: High—attack closely resembled typical usage flow and targeted wallets with pre-approved permissions
- **Root_cause**: Missing input validation in anySwapOutUnderlyingWithPermit()—the contract didn’t enforce token allowlist checks, enabling misuse of permit() on arbitrary tokens
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-multichain-hack-january-2022/) 
