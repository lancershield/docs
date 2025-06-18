# Velodrome Frontend/DNS Attack 

- **Project**: Velodrome 
- **Exploit_type**: DNS Hijacking → Frontend Phishing
- **Loss**: $250,000 
- **Entry_point**: Browser‑served frontend from attacker‑controlled domains
- **Exploit_vector**: Attackers gained control of the domain registrar via social engineering, changed nameservers, and served malicious UI scripts that prompted users to approve transactions draining assets.
- **Severity**: Critical
- **Attack_steps**:
    - Attackers used social engineering to compromise Velodrome’s and Aerodrome’s domain registrar account, bypassing 2FA.
    - Nameservers were changed to routes under attacker control.
    - Users visiting the legitimate URLs were redirected to cloned frontends containing malicious JavaScript.
    - The malicious UI prompted users to connect wallets and sign transactions approving transfers and token spends.
    - Funds (approx. up to $250K) were stolen from wallets that approved these malicious requests.
    - Velodrome confirmed the incident, issued warnings, and began domain recovery procedures and user outreach. 
- **Impact**: Approx. $250K stolen across multiple chains; no internal wallets or protocol funds were affected.
- **Exploitability**: High
- **Root_cause**: Lack of registrar-level security and monitoring—compromised domain ownership led to cloned UIs capable of phishing unsuspecting users.
- **Resource**:[Link](https://medium.com/@VelodromeFi/11-29-2023-incident-report-92865dceb757)