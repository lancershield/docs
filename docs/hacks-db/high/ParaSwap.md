# ParaSwap Augustus V6 Bug 

- **Project**: ParaSwap
- **Exploit_type**: Reentrancy / Callback Handling Flaw
- **Loss**: ~$24,000 (stolen via MEV attackers)
- **Entry_point**: uniswapV3SwapCallback() in the Augustus V6 smart contract
- **Exploit_vector**: The callback function didn’t verify the caller, enabling MEV-enabled bots to exploit user allowances and drain funds via a crafted Uniswap V3 callback.
- **Severity**: High
- **Attack_steps**:
    - Augustus V6 was launched on March 18, 2024 to optimize gas usage.
    - The callback did not check whether the caller was the trusted Uniswap V3 pool.
    - Malicious MEV bots created fake pools to invoke the callback and exploit user allowances.
    - About five exploit transactions occurred before the bug was caught.
    - Attackers successfully drained ~$24,000 from user wallets holding an approval to Augustus V6.
    - ParaSwap paused the V6 API and reverted to V5 while security teams and whitehats secured ~$3.4M from vulnerable addresses. 
- **Impact**: ~$24K stolen from user wallets; ~$3.4M recovered through whitehat intervention
- **Exploitability**: High
- **Root_cause**: Missing authorization in callback logic—no validation of msg.sender in uniswapV3SwapCallback() allowing malicious contract calls
- **Resource**:[Link](https://cointelegraph.com/news/paraswap-return-crypto-critical-smart-contract-vulnerability)