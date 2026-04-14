[<img alt="Contrast Security" src="https://www.contrastsecurity.com/hubfs/contrast-web-platform--2025/images/logos/contrast-logo--full-color.png"  />](https://www.contrastsecurity.com/)

# Contrast ADR — Google Security Operations

---

## Overview

This repository hosts the official YARA-L detection rules for the Contrast Security ADR integration with Google Security Operations.

- [Google Security Operations integration documentation](https://docs.cloud.google.com/chronicle/docs/reference/partner-hosted-siem-integrations)
- [Contrast Security ADR integration documentation](https://docs.contrastsecurity.com/en/google-security-operations-with-adr-317284.html)

### Install Detection Rules

You can create URL rules in the Google Secops console by navigating to Detection->Rules & Detections->Rules->Create New Rule. We've provided YARA-L rules in the `detection_rules` directory to import. After creating the rules, make sure to activate them and set them to a run frequency of at least once daily

You can also install our correlation rules useing the new [Google SecOps CLI](https://github.com/google/secops-wrapper/blob/main/CLI.md) using `secops rule create --file path/to/file.yaral`. To install all of our correlation rules

```
for f in detection_rules/*; do
    secops rule create --file $f
done
```

---

## Detection Rules

All rules are authored by Contrast Security and use **Contrast ADR** as the primary signal source. Contrast ADR instruments the application runtime to definitively determine when an attack payload reaches a dangerous function — not just attempted, but actually executed.

---

### `contrast_adr_exploited_attack_in_production`

| Field | Value |
|---|---|
| Severity | CRITICAL |
| Priority | HIGH |
| MITRE Tactics | Execution, Impact |
| MITRE Techniques | T1059, T1190 |
| Data Source | Contrast ADR |

Fires when Contrast ADR confirms an application-layer attack was **successfully exploited** (not just attempted) in a `PRODUCTION` environment. ADR instruments the application runtime and can definitively detect when a payload reaches a dangerous function (database query, system command, file operation, etc.). The production filter reduces noise from development and testing environments where exploitation may be expected.

---

### `contrast_adr_exploited_attack_corroborated_by_edr`

| Field | Value |
|---|---|
| Severity | CRITICAL |
| Priority | HIGH |
| MITRE Tactics | Execution, Impact |
| MITRE Techniques | T1059, T1190 |
| Correlation Window | 30 minutes |
| Data Sources | Contrast ADR, CrowdStrike Falcon, SentinelOne, Falco, or any EDR in `%contrast_edr_product_names` |

Correlates a confirmed ADR exploit with an EDR detection on the **same hostname** within a 30-minute window. When ADR confirms an application-layer attack (e.g. command injection, JNDI injection) was exploited, and an EDR independently detects suspicious process, file, or network activity on the same host, this provides high-confidence evidence the exploit led to system-level compromise.

> **Note:** In Kubernetes environments, ADR `target.hostname` is the pod/app hostname while EDR `principal.hostname` may be the node name — adjust the Contrast agent `server.name` or add a shared label to align.

---

### `contrast_adr_exploited_attack_corroborated_by_waf`

| Field | Value |
|---|---|
| Severity | CRITICAL |
| Priority | HIGH |
| MITRE Tactic | Initial Access |
| MITRE Technique | T1190 |
| Correlation Window | 5 minutes |
| Data Sources | Contrast ADR, ModSecurity, AWS WAF, Cloudflare, Imperva, or any WAF in `%contrast_waf_product_names` |

Correlates a confirmed ADR exploit with a WAF detection matching the **same source IP and URL path** within a 5-minute window. Particularly valuable when the WAF operates in detection-only mode — ADR confirms the WAF alert was a real exploit, providing multi-layer evidence of a genuine attack.

---

### `contrast_adr_sql_injection_with_dlp_alert`

| Field | Value |
|---|---|
| Severity | CRITICAL |
| Priority | HIGH |
| Risk Score | 99 |
| MITRE Tactics | Collection, Exfiltration |
| MITRE Techniques | T1530, T1567 |
| Correlation Window | 1 hour |
| Data Sources | Contrast ADR, Google Cloud DLP, Microsoft Purview, or any DLP in `%contrast_dlp_product_names` |

Correlates a confirmed ADR SQL injection exploit with a DLP alert on the **same host** within a 1-hour window. Provides evidence that a SQL injection attack resulted in actual sensitive data access or exfiltration.

> **Note:** This rule has an inherently weaker join key (hostname only) compared to the WAF and EDR correlation rules. Consider complementing with a SOAR playbook for API-based enrichment.

---

### `contrast_adr_incident_with_attack_evidence`

| Field | Value |
|---|---|
| Severity | HIGH |
| Priority | HIGH |
| MITRE Tactics | Execution, Impact |
| MITRE Techniques | T1059, T1190 |
| Correlation Window | 24 hours |
| Data Source | Contrast ADR |

Joins Contrast ADR **incident** events (the platform's high-level case summaries) with their underlying **attack events** using a shared `incident_id`. Enriches SOC alerts by combining the incident summary, recommended actions, runbooks, and all granular attack evidence — payloads, stack traces, HTTP request details, and the exact database query or system command affected — into a single detection. One incident maps to one or more attack events (1:N relationship).
