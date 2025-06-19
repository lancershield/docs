# Curio Smart Contract Access-Control Exploit

- **Project**: Curio
- **Exploit_type**: Access-Control Logic Vulnerability in Governance / Token Minting
- **Loss**: ~$16 million 
- **Entry_point**: Governance mint and privilege execution logic within Curio DAO contracts (CGT token minting function)
- **Exploit_vector**: Attacker acquired a small number of governance tokens and used flawed access control to elevate privileges, allowing them to mint 1 billion CGT (~$16 M)
- **Severity**: Critical
- **Attack_steps**:
    - Attacker obtained a minimal amount of Curio governance tokens (CGT) to meet the voting threshold.
    - Exploited a logic flaw in the governance contract that failed to enforce strict access control during mint function execution.
    - Submitted a malicious governance proposal using limited privileges to execute on-chain mint logic.
    - Proposal passed, triggering unauthorized minting of 1 billion CGT.
    - Attacker transferred minted tokens to external addresses, converting them to other assets.
- **Impact**: Illicit minting of 1 billion CGT ($16 M); token supply inflated; poses long-term economic damage
- **Exploitability**: High — governance logic allowed mint without proper authorizations
- **Root_cause**: Weak access control in smart contract's minting function and oversight in governance privilege validation
- **Resource**:[Link](https://cointelegraph.com/news/curio-smart-contract-exploit-hacker-mints-1-billion-tokens)
