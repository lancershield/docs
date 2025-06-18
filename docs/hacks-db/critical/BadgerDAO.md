# BadgerDAO Frontend Attack 

- **Project**: BadgerDAO
- **Exploit_type**: Frontend Injection (Phishing via UI compromise)
- **Loss**: ~$120 million
- **Entry_point**: BadgerDAO frontend application (injected malicious script)
- **Exploit_vector**: The attacker gained access to Badger's Cloudflare frontend infrastructure and injected malicious JavaScript into the UI. This script tricked users into signing malicious transactions, which transferred funds to the attacker.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker compromised BadgerDAO’s frontend hosting provider (Cloudflare config or API keys).
    - They injected a malicious script into the frontend code served to users.
    - When users interacted with the Badger dApp, the injected script triggered wallet prompts.
    - Unsuspecting users signed malicious approvals or transfer calls.
    - The attacker used these permissions to drain funds from user wallets.
- **Impact**: ~$120M in user wallets drained, including high-profile DeFi whales and institutional addresses.
- **Exploitability**: High
- **Root_cause**: Lack of frontend integrity checks and compromised deployment pipeline allowed malicious JS injection — no runtime verification of UI integrity or transaction origin.
- **Resource**:[Link](https://www.vidma.io/blog/the-badger-hack-a-120-million-lesson-in-front-end-security)