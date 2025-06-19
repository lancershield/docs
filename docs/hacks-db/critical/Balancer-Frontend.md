# Balancer Frontend DNS Hijack 

- **Project**: Balancer
- **Exploit_type**: DNS Hijacking → Frontend Phishing
- **Loss**: ~$238,000–$364,000 
- **Entry_point**: balancer.fi and app.balancer.fi front-end
- **Exploit_vector**: Attackers used social engineering to compromise Balancer’s EuroDNS registrar. They changed nameservers to serve a malicious frontend, prompting wallet approvals and draining tokens.
- **Severity**: Critical
- **Attack_steps**:
    - Attackers conducted social engineering on Balancer’s domain registrar (EuroDNS) 
    - Nameserver records for balancer.fi were modified to redirect to attacker-controlled servers.
    - A malicious clone of the Balancer UI injected scripts to trigger wallet signature requests (transferFrom, permit) when users interacted.
    - Victims reviewing the UI unknowingly signed malicious transactions approving token allowances.
    - The attacker executed draining transactions, moving ~$238K initially (later estimated up to $364K) to mixer addresses 
    - Balancer DAO responded with alerts, domain recovery actions, and began deprecating the .fi domain 
- **Impact**: $238K–$364K stolen; Balancer domains were compromised; user trust damaged and protocol temporarily inoperative.
- **Exploitability**: High
- **Root_cause**: Insecure DNS management—weak registrar security allowed frontend hijack; absence of domain monitoring and DNSSEC.
- **Resource**:[Link](https://decrypt.co/197953/balancer-frontend-hit-by-dns-attack-over-250k-stolen)