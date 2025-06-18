# Blur NFT Bid Re‑acceptance Bug 

- **Project**: Blur (Ethereum NFT Marketplace)
- **Exploit_type**: Bid Acceptance Logic Flaw
- **Loss**: Unknown (some users accepted unintended old bids; amounts likely small)
- **Entry_point**: Marketplace contract’s bid acceptance function
- **Exploit_vector**: The bug allowed previously canceled NFT bids to be re‑accepted, enabling buyers or bots to purchase NFTs at outdated prices.
- **Severity**: High 
- **Attack_steps**:
    - Blur deployed a new bidding system without properly invalidating historical bid records.
    - A user had canceled a bid months earlier at a price much higher than the current listing.
    - Attacker or automated tooling triggered the acceptance of this old, high-value bid.
    - The marketplace contract executed the sale at the outdated bid price.
    - NFT transferred to the attacker (or accepted bid owner), while the seller received less than expected.
    - Blur temporarily disabled bid acceptance, refunded impacted users, and fixed the logic. 
- **Impact**: Multiple cases of users unintentionally accepting old bids — some reportedly lost 70 ETH (~$120K) on single NFTs. 
finance.yahoo.com
- **Exploitability**: Medium
Root_cause: Marketplace failed to remove or update historic bid entries on bid cancellation—accepted bids relied on stale data validation.
- **Resource**:[Link](https://www.web3isgoinggreat.com/single/blur-nft-platform-bug-allows-old-bids-to-be-accepted) 

