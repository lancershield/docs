# Linode Infrastructure Hack 

- **Project**: Linode (Cloud Hosting Provider)
- **Exploit_type**: Infrastructure Breach via Social Engineering & Internal Tool Exploit
- **Loss**: ~$250,000 
- **Entry_point**: Linode’s internal customer support interface
- **Exploit_vector**: The attacker gained unauthorized access to Linode’s backend support system and used it to reset credentials and access VPS instances of Bitcoin companies and individual holders.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker used social engineering or insider compromise to gain access to Linode’s internal admin panel.
    - Identified and targeted accounts associated with Bitcoin services and wallets (including Bitcoinica, Slush, and individual holders).
    - Used Linode’s backend to reset root credentials on targeted VPS servers.
    - Accessed the running virtual machines and extracted hot wallet private keys or directly transferred BTC.
    - Consolidated and moved BTC to attacker-controlled wallets.
    - Incident triggered major backlash over Linode’s lack of access controls and audit trails.
- **Impact**: ~$250,000 in Bitcoin stolen from multiple early crypto entities; some companies were permanently crippled.
- **Exploitability**: High
- **Root_cause**: Inadequate internal access control and lack of audit logging on privileged customer support tools, allowing complete administrative takeover without detection.
- **Resource**:[Link](https://www.infosecurity-magazine.com/news/linode-web-hosting-hack-used-adobe-coldfusion/)
