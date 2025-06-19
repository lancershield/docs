# Raccoon Stealer Malware Campaign 

- **Project**: Raccoon Stealer (aka Racoon/Mohazo)
- **Exploit_type**: Infostealer / Credential & Crypto Wallet Data Theft
- **Loss**: ~$13 million
- **Entry_point**: Infection via phishing, cracked software, exploit kits
- **Exploit_vector**: Delivered through phishing links or cracked software installers; once executed, Raccoon grabs private keys, seed phrases, and clipboard data for cryptocurrency wallets and exfiltrates them to C2 servers
- **Severity**: High
- **Attack_steps**:
    - Malware is purchased as a service and distributed via phishing or fake cracked apps 
    - Victim downloads a malicious installer (often disguised as cracked software); executes the Raccoon binary
    - Raccoon scans system storage, browser data, and wallet files (e.g., MetaMask, Atomic Wallet) 
    - Extracts credentials, wallet private keys, cookies, credit card details, and clipboard data
    - Periodically sends stolen data via HTTP POST to attacker-controlled servers, often via Google Cloud infrastructure 
    - Malware may drop secondary payloads (clippers, clipboard hijackers) and establish persistence via scheduled tasks
- **Impact**: Numerous user crypto wallets compromised, resulting in theft of funds across exchanges and personal accounts; estimated losses range in the low to mid six figures per campaign
- **Exploitability**: High – Distributed as a full malware-as-a-service kit requiring only execution by the victim
- **Root_cause**: User execution of untrusted code; lack of endpoint protection; victims download malicious files disguised as legitimate software
- **Resource**:[Link](https://www.bitdefender.com/blog/hotforsecurity/raccoon-malware-aims-to-steal-credentials-of-people-who-use-popular-apps/)