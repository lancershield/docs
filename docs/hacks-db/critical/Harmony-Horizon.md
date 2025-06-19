# Harmony Horizon Bridge Exploit 

- **Project**: Harmony (Horizon Bridge)
- **Exploit_type**: Private Key Compromise of Validator Key
- **Loss**: ~$100 million
- **Entry_point**: Multisig bridge contract controlled by a 2-of-5 validator signature system
- **Exploit_vector**: Attackers obtained private keys for at least two validator-controlled multisig addresses, allowing them to authorize and execute unauthorized cross-chain withdrawals
- **Severity**: Critical
- **Attack_steps**:
    - Attackers compromised at least 2 of the 5 multisig validator addresses (likely via key leakage or social engineering)
    - They crafted and signed withdrawal transactions across Ethereum and BNB chains
    - Executed 14 cross-chain withdrawal transactions, draining ETH, USDT, USDC, DAI, BNB, and WBTC from the bridge
    - Swapped stolen assets for ETH on decentralized exchanges
    - Converted a significant portion (~84,000 ETH) through Tornado Cash into 148 withdrawal wallets 
    - Bridge was temporarily suspended by Harmony; they offered a $1 million bounty for recovery and launched investigations
- **Impact**: ~$100M in multisig-protected bridge funds drained and laundered — user cross-chain assets locked or compromised
- **Exploitability**: High
- **Root_cause**: Insufficient validator key security and dependence on low-threshold multisig (2/5); absence of real-time monitoring and rate-limiting allowed full loss to occur repeatedly
- **Resource**:[Link](https://www.elliptic.co/blog/analysis/over-1-billion-stolen-from-bridges-so-far-in-2022-as-harmony-s-horizon-bridge-becomes-latest-victim-in-100-million-hack/hss_channeltw-1344645140)

