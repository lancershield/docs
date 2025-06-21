# EtherDelta DNS Attack

- **Project**: EtherDelta
- **Exploit_type**: DNS Hijacking / Frontend Compromise
- **Loss**: ~$305,000
- **Entry_point**: Domain Name System (DNS) of EtherDelta frontend
- **Exploit_vector**: Attacker hijacked the DNS records, redirecting users to a malicious frontend that mimicked EtherDelta and captured private keys.
- **Severity**: Critical
- **Attack_steps**:
    - Gained unauthorized access to EtherDelta’s DNS provider
    - Modified DNS records to point to a fake website
    - Deployed a phishing site with a replica of EtherDelta UI
    - Captured users’ private keys as they interacted with the malicious site
    - Drained wallets by submitting unauthorized transactions
- **Impact**: ~$305,000 worth of assets stolen from users who unknowingly accessed the phishing site
- **Exploitability**: High — due to centralized DNS infrastructure
- **Root_cause**: Lack of DNS-level security; no DNSSEC, poor registrar protection
- **Resource**:[Link](https://www.bitcoininsider.org/article/12695/etherdeltas-dns-hacked-website-replaced-hackers-duplicate-steal-funds)

