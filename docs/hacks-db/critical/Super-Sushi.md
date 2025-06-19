# Super Sushi Samurai Token Mint Exploit 

- **Project**: Super Sushi Samurai (SSS) on Blast L2
- **Exploit_type**: Smart Contract Mint Logic Bug
- **Loss**: ~$4.6 million
- **Entry_point**: mint() function in the SSS token contract
- **Exploit_vector**: A bug allowed users to double their own balance on self-transfer, enabling unlimited token minting before dumping into liquidity pools.
- **Severity**: Critical
- **Attack_steps**:
    - Deployed SSS token contract with a flawed mint() or transfer() logic.
    - Attacker transferred tokens to themselves, doubling their balance.
    - Repeated self-transfers multiple times to exponentially increase token holdings.
    - Sold newly minted tokens directly into the SSS liquidity pool.
    - Dumped tokens on the open market, collapsing token price by over 99%.
    - Project team engaged with attacker (claimed white-hat intent to reimburse victims).
- **Impact**: ~$4.6M drained from token launch; price crash; severe liquidity damage.
- **Exploitability**: High (bug replicated via simple self-transfers)
- **Root_cause**: Faulty mint/transfer logic that increased balance before deduction — no proper subtraction calculation when transferring to same address.
- **Resource**:[Link](https://www.coindesk.com/business/2024/03/21/newly-issued-gaming-token-exploited-on-blast-with-46m-drained)