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

All rules use Contrast ADR as the primary signal source.

---

### `contrast_adr_exploited_attack_in_production`

Fires on confirmed ADR exploits scoped to production environments.

---

### `contrast_adr_exploited_attack_corroborated_by_edr`

Correlates a confirmed ADR exploit with an EDR alert on the same host, indicating host-level compromise following an application exploit.

---

### `contrast_adr_exploited_attack_corroborated_by_waf`

Correlates a confirmed ADR exploit with a WAF alert on the same source IP and URL, corroborating perimeter and application-layer detections.

---

### `contrast_adr_sql_injection_with_dlp_alert`

Correlates a confirmed ADR SQL injection exploit with a DLP alert on the same host, indicating potential data exfiltration.

---

### `contrast_adr_incident_with_attack_evidence`

Joins ADR incident summaries with their underlying attack events for consolidated SOC triage.
