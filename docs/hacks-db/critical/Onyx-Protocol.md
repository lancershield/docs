# Onyx Protocol Price/Interest Exploit 

- **Project**: Onyx Protocol
- **Exploit_type**: Flash Loan Price Manipulation + Rounding Bug (Compound v2 fork)
- **Loss**: ~$3.8 million 
- **Entry_point**: Mint/Redeem functions in new VUSD and Lending Markets
- **Exploit_vector**: The attacker leveraged flash loans to mint/redeem via empty VUSD markets, triggering exchange rate errors, combined with NFT liquidation misvalidation to extract funds.
- **Severity**: Critical
- **Attack_steps**:
    - Created an empty VUSD market without initial liquidity, then donated tokens to seed it. 
    - Used flash loans to mint large VUSD through this low-liquidity market, exploiting rounding errors in market exchange rates. 
    - Redeemed VUSD for underlying assets at manipulated rates, draining funds.
    - Took advantage of improper validation in NFTLiquidation contract to increase liquidation rewards, compounding extraction. 
- **Impact**: Protocol drained of ~4.1M VUSD, 7.35M XCN, 0.23 WBTC, 5 k DAI, 50 k USDT; ~$3.8M total 
- **Exploitability**: High
- **Root_cause**: Inherited rounding error in Compound v2 fork under low-liquidity conditions and flawed input validation in liquidation contract
- **Resource**:[Link](https://cointelegraph.com/news/onyx-protocol-exploited-second-time-3-8m-via-known-bug)
