# Transit Swap Exploit 

- **Project**: Transit Swap
- **Exploit_type**: Arbitrary External Call via User Input
- **Loss**: ~$21 million
- **Entry_point**: swap() function in the Transit Swap router contract
- **Exploit_vector**: The router contract allowed users to specify arbitrary call targets and calldata, enabling attackers to trick the contract into sending user funds to malicious contracts.
- **Severity**: Critical
- **Attack_steps**:
    - Transit Swap deployed a router contract that allowed arbitrary external calls based on user input.
    - The attacker crafted a malicious payload targeting this arbitrary call feature.
    - Passed their own contract address and calldata as input to the router’s `swap()` function.
    - The router blindly executed the call, transferring user-approved tokens directly to the attacker’s address.
    - Multiple users were impacted as the attacker repeatedly triggered the exploit during active transactions.
    - Around $21 million in assets were stolen from Transit Swap users.
- **Impact**: ~$21M worth of user assets (including USDT, BUSD, ETH, etc.) were stolen across multiple chains.
- **Exploitability**: High
- **Root_cause**: Failure to sanitize and restrict external call targets in the router’s logic—user-controlled inputs were directly used to trigger call() operations without checks or allowlists.
- **Resource**:[Link](https://cryptonews.com/news/hacker-exploits-transit-swap-steals-23-million-heres-everything-you-need-know/)