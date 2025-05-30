# 🟠 High Severity

## 🧾 Definition:

- Represents major vulnerabilities that could be exploited under certain conditions, resulting in financial loss, protocol disruption, or security bypass. These require urgent attention but may have contextual safeguards.

## 🔐 Key Characteristics:

- **Conditional Exploitability**: Requires attacker effort, setup, or timing, but is practically exploitable.
- **Significant Loss Risk**: Can drain user rewards, bypass role checks, or misconfigure sensitive parameters.
- **Protocol-Specific Impact**: Often impacts tokenomics, governance, vaults, or price-sensitive flows.
- **External Dependency Abuse**: May stem from or impact third-party contracts, oracles, or integrations.
- **Elevated Trust Risk**: Weakens user trust or protocol integrity, even without direct loss.
- **Can Be Mitigated On-Chain**: Sometimes manageable through upgrades, role transfers, or parameter changes.

## 🚨 Required Response:

- **Patch Required Before Production**: Must be fixed before going live.
- **On-Chain Mitigation (if live)**: Use timelocks, pause guards, or reconfiguration if fix isn’t immediate.
- **Monitoring & Alerts**: Track usage of affected components until resolution.

