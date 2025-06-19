# Curve Finance Frontend (DNS) Hijack 

- **Project**: Curve Finance
- **Exploit_type**: DNS Hijacking → Frontend Phishing
- **Loss**: ~ $570,000 
- **Entry_point**: DNS record manipulation of the curve.fi domain
- **Exploit_vector**: Attackers gained access to the domain registrar, rerouted the site to a malicious frontend that mimicked the UI and invoked wallet-draining transactions via phishing scripts.
- **Severity**: Critical
- **Attack_steps**:
    - Gained access to the registrar for curve.fi, changing DNS to point to attacker-controlled IPs. 
    - Redirected user traffic to a cloned frontend containing malicious JavaScript designed to prompt wallet transactions.
    - Users attempting legitimate interactions saw deceptive transaction prompts, potentially authorizing token transfers.
    - Blockaid detected suspicious activity and notified Curve, prompting a public warning. 
    - Curve responded by freezing the compromised domain, shifting users to a safe domain (curve.finance), and engaging the registrar to secure DNS. 
    - No on-chain exploit occurred during the 2025 hijack, but user trust was significantly shaken due to phishing potential.
- **Impact**: No direct funds stolen in 2025, but significant risk exposure; previous 2022 hijack led to ~$570K loss. 
- **Exploitability**: High
- **Root_cause**: Insecure domain management—lack of robust registrar security (multi-factor authentication and DNSSEC) allowed redirection to malicious frontend.
- **Resource**:[Link](https://cointelegraph.com/explained/what-is-dns-hijacking-how-it-took-down-curve-finances-website)