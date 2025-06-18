# 🟠 High Severity

## 🧾 Definition:

High severity hacks exploit critical parts of the smart contract but may require specific conditions or partial access to execute. While they may not immediately drain all funds, they pose significant risk to the protocol’s integrity, economic model, or user trust. If unaddressed, they often escalate to critical vulnerabilities.

## 🔐 Key Characteristics:

- **Conditional Exploitability**: Attack path exists but may require timing, governance delays, or privileged access.
- **Partial Fund Exposure**: Only a subset of user funds or protocol assets is at risk unless compounded.
- **Economic Disruption**: Alters core economic mechanisms—e.g., oracle manipulation, front-running rewards, or interest rate exploits.
- **Governance Risk**: Vulnerability could be weaponized through malicious proposals or upgrade delays.
- **Dependency Exploits**: Rooted in external dependencies such as token contracts, math libraries, or protocol integrations.
- **Visible on Explorers**: May show abnormal usage patterns or suspicious approvals before full-blown exploitation.
- **Containable Scope**: Affected contracts can usually be isolated without shutting down the entire system.

## 🚨 Required Response:

- **Hotfix Preparation**: Draft and simulate remediation plan for rapid upgrade or redeploy.
- **Partial Pauses**: Pause only affected functions (e.g., minting, claiming) while leaving read-only or safe functions active.
- **On-Chain Monitoring**: Set up heuristics or bots to detect abnormal usage related to the exploit vector.
- **Responsible Disclosure**: Notify auditors, security communities, and relevant integrations before public announcement.
- **User Communication**: Inform users of partial risk exposure and recommended actions (e.g., unstaking, withdrawing).
- **Re-Audit**: All patches must undergo at least one independent audit before full reactivation.