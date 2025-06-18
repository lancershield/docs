# Snowdog DAO Rug Pull 

- **Project**: Snowdog DAO Rug Pull
- **Exploit_type**: Insider Liquidity Withdrawal / Rug Pull
- **Loss**: ~$10 million
- **Entry_point**: Custom AMM contract used for executing the buyback event
- **Exploit_vector**: The Snowdog team created a custom AMM to conduct a one-time buyback event. Only a whitelisted insider contract was allowed to execute trades before the liquidity was pulled, trapping retail users.
- **Severity**: Critical
- **Attack_steps**:
    - Snowdog announced a one-time buyback event using a custom-built AMM instead of using Trader Joe (a popular DEX).
    - Users were led to believe they could sell their SDOG tokens during this event at a premium.
    - Team seeded the AMM with ~$10M in liquidity and whitelisted only their own contract to access the AMM during the buyback.
    - As the buyback began, the insider contract executed trades and extracted all liquidity.
    - Normal users who tried to sell SDOG found the AMM drained or inaccessible, causing the SDOG price to collapse by over 90%.
    - Team later claimed the action was part of the experiment, sparking backlash.
- **Impact**: ~$10M in liquidity drained, SDOG token lost over 90% of its value instantly.
- **Exploitability**: High
- **Root_cause**: Team used a custom AMM with a whitelisted contract and failed to disclose critical design mechanics, enabling a controlled drain of liquidity under false pretenses.
- **Resource**:[Link](https://pexx.com/chaindebrief/snowdog-avalanche-rugpull/)