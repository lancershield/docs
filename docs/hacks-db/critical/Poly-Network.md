# Poly Network Cross-Chain Exploit 

- **Project**: Poly Network
- **Exploit_type**: Cross-Chain Signature Validation Flaw
- **Loss**: ~$611 million
- **Entry_point**: verifyHeaderAndExecuteTx() in the cross-chain bridge contract
- **Exploit_vector**: The attacker manipulated a flaw in the cross-chain message verification logic, allowing them to forge a message that gave them control over funds on multiple chains.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker reviewed the contract responsible for validating cross-chain transactions.
    - They discovered that the contract trusted data passed in the verifyHeaderAndExecuteTx() function without verifying the signer properly.
    - They crafted a malicious cross-chain message that assigned themselves ownership of funds on Ethereum, BSC, and Polygon.
    - Called the execution function with the forged message to trigger unauthorized fund transfers.
    - Extracted over $600M in assets across multiple blockchains in a single transaction per chain.
- **Impact**: ~$611M worth of ETH, BNB, USDC, DAI, and other tokens stolen from cross-chain bridge contracts.
- **Exploitability**: High
- **Root_cause**: Insecure trust model and flawed validation in cross-chain communication logic, where message authenticity wasn’t properly cryptographically enforced.
- **Resource**:[Link](https://www.certik.com/resources/blog/poly-network-incident-analysis)

