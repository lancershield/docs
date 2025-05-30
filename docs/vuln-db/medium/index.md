# 🟡 Medium Severity

## 🧾 Definition:

- Medium vulnerabilities do not lead to direct loss but can be abused under specific edge cases, impact economic fairness, or weaken system robustness. Often involve misconfigurations or incomplete checks.

## 🔐 Key Characteristics:

- **Exploitability Requires Context**: Often needs specific timing, user error, or third-party behavior.
- **Impacts Protocol Fairness or Accuracy**: Affects reward math, staking behavior, or accounting.
- **Mild Trust Erosion**: Reduces trust if exploited, even if no financial loss occurs.
- **Integration Risk**: May break assumptions for dApps, wallets, or external tools.
- **Inconsistency or Drift**: Causes logic to degrade over time without proper resets or corrections.
- **Fixable With Minimal Impact**: Usually fixable with a small contract update or config patch.

## 🚨 Required Response:

- **Fix Strongly Recommended**: Should be addressed before production, but may be queued if documented.
- **Add Monitoring or Failsafes**: Track drift or limit access to reduce exposure while patching.
- **Mention in Public Docs**: If unpatched in production, must be disclosed transparently.



