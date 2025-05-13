# LancerShield Severity Framework (LSF)

The LancerShield Severity Framework (LSF) provides a structured, consistent, and explainable way to rate vulnerabilities discovered during smart contract audits. It enables better prioritization, automation, and trust in the audit process.

---

## Severity Levels

| Code  | Label         | Meaning                                                              |
| ----- | ------------- | -------------------------------------------------------------------- |
| **C** | Critical      | Direct fund loss, contract destruction, or irreversible control loss |
| **H** | High          | Significant theft, denial-of-service, or unauthorized state changes  |
| **M** | Medium        | Conditional exploit requiring specific inputs or edge case setup     |
| **L** | Low           | Minor issues, unlikely to be exploited or low impact                 |
| **I** | Informational | No impact - readability, style, or developer notes                   |
| **G** | Gas           | Gas inefficiencies only, no security or functional risk              |

---

## Scoring Criteria (Axis)

Each issue is scored across five axis from 0–5, with weighted importance.

| Axis               | Weight | Scope                                            |
| ------------------ | ------ | ------------------------------------------------ |
| **Impact**         | 40%    | Severity of damage if exploited                  |
| **Exploitability** | 25%    | Ease of triggering the bug                       |
| **Reachability**   | 15%    | Can the code path actually be reached?           |
| **Complexity**     | 10%    | Setup effort or attacker sophistication required |
| **Detectability**  | 10%    | How likely is this to be missed during review?   |

> 🔒 Impact or Reachability of 0 auto-downgrades severity to Informational.

---

## Severity Calculation Logic

### Weighted Score → Severity Mapping:

| Final Score | Severity          |
| ----------- | ----------------- |
| 4.5 - 5.0   | Critical (C)      |
| 3.5 - 4.49  | High (H)          |
| 2.5 - 3.49  | Medium (M)        |
| 1.5 - 2.49  | Low (L)           |
| 0 - 1.49    | Informational (I) |

---

## Severity Level Scope

### Critical (C)

- Scope: Catastrophic impact (e.g. loss of control, fund drains)
- Must be fixed before deployment
- Color Code: <span style="display:inline-block;width:14px;height:14px;background-color:#D32F2F;border-radius:3px;margin-right:6px;"></span> `#D32F2F`

### High (H)

- Scope: Major financial or functional impact
- Should be fixed before mainnet
- Color Code: <span style="display:inline-block;width:14px;height:14px;background-color:#F57C00;border-radius:3px;margin-right:6px;"></span> `#F57C00`

### Medium (M)

- Scope: Exploitable with effort, medium-risk
- Fix recommended; acceptable in staging
- Color Code: <span style="display:inline-block;width:14px;height:14px;background-color:#FBC02D;border-radius:3px;margin-right:6px;"></span> `#FBC02D`

### Low (L)

- Scope: Low impact or unlikely execution
- Fix if convenient
- Color Code: <span style="display:inline-block;width:14px;height:14px;background-color:#388E3C;border-radius:3px;margin-right:6px;"></span> `#388E3C`

### Informational (I)

- Scope: No security or runtime effect
- No fix needed, but improves readability
- Color Code: <span style="display:inline-block;width:14px;height:14px;background-color:#1976D2;border-radius:3px;margin-right:6px;"></span> `#1976D2`

### Gas (G)

- Scope: Optimizations only
- No functional or security effect
- Color Code: <span style="display:inline-block;width:14px;height:14px;background-color:#616161;border-radius:3px;margin-right:6px;"></span> `#616161`

---

## Override + Audit Logging

Auditors may override the computed severity with justification. All overrides are recorded with user ID, timestamp, and cryptographic hash for transparency.

> The LancerShield Severity Framework (LSF) is still under refinement. Additional override logic and edge-case rules are being continuously evaluated based on real-world audit data.
