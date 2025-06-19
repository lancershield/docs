# Uniswap v1 & imBTC Reentrancy Exploit

- **Project**: Uniswap v1 with imBTC token
- **Exploit_type**: Reentrancy attack via ERC‑777 tokensToSend hook
- **Loss**: ~$300,000 worth of ETH 
- **Entry_point**: tokenToEthSwapInput() function in Uniswap v1 exchange contract (part of token-to-ETH swaps) 
- **Exploit_vector**: Attacker used ERC‑777 token standard’s tokensToSend() callback to re-enter the swap function before reserves updated—enabling repeated overpriced swaps
- **Severity**: Critical 
- **Attack_steps**:
    - Attacker registered a malicious tokensToSend() hook via ERC‑1820 registry 
    - Initiated tokenToEthSwapInput() to swap imBTC for ETH.
    - Uniswap sent ETH before updating token reserves.
    - During the pending token transferFrom(), the malicious hook re-entered tokenToEthSwapInput().
    - Each re-entry used stale reserves to extract more ETH for the same tokens.
- **Impact**: Full drain of about $300k ETH from Uniswap imBTC pool; liquidity pools and related services (e.g., dForce Lendf.Me) also affected 
- **Exploitability**: High 
- **Root_cause**: Swapped ETH was sent before state (token reserve) was updated. Combined with ERC‑777’s callback mechanism, this violated Checks‑Effects‑Interactions pattern and enabled reentrancy 
- **Resource**:[Link](https://milkroad.com/news/imbtc-uniswap-hack/)