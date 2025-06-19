# 🟡 Medium Severity

## 🧾 Definition:

Medium severity hacks exploit weaknesses that are context-dependent or require specific contract states to be effective. These exploits typically affect functionality, business logic, or partial access control, and may not always lead to immediate loss of funds, but they can erode trust, cause misbehavior, or serve as precursors to more severe breaches.

## 🔐 Key Characteristics:

- **State-Dependent Exploitability**: Requires specific on-chain conditions (e.g., unlocked rewards, certain block timing, user roles).
- **Misbehavior over Theft**: Impacts protocol behavior such as payout delays, logic inversion, or fee bypass—rather than direct fund loss.
- **Exploit via Misuse**: Often abused by manipulating intended behaviors like emergency withdrawal, partial refunds, or fallback functions.
- **Escalation Potential**: If chained with other issues, may evolve into High or Critical severity (e.g., combined with a price oracle flaw).
- **Logical Errors**: Includes unchecked return values, incorrect tier validation, or missed edge case handling.
- **Visible in Logs**: Anomalies are often noticeable through logs, state variables, or excessive gas consumption during edge-case testing.
- **Fixable without Full Shutdown**: Typically requires a version bump, logic patch, or updated validation check—without pausing the whole protocol.

## 🚨 Required Response:

- **Patch Ready in Dev Branch**: Implement and test a patch in a staging environment immediately.
- **Feature Flagging**: Temporarily disable or restrict access to the affected function using feature toggles or modifiers.
- **Regression Testing**: Verify other flows that may share assumptions or data paths.
- **Targeted Disclosure**: Notify project partners (e.g., aggregators, DEXs, vaults) of the functional bug and any mitigation steps.
- **Optional User Action**: Inform affected users only if manual action (e.g., claim before fix) is beneficial or safe.
- **Follow-Up Audit**: Include in next audit cycle with emphasis on state-dependent logic and test coverage gaps.

