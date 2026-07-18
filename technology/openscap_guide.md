# OpenSCAP — NIST-Certified SCAP Compliance Scanning & Vulnerability Assessment

**Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB, Singapore
**Path:** `technology/openscap_guide.md` | **Version:** 1.0 — July 2026

---

## 1. Overview & Certification

**OpenSCAP** is the open-source reference implementation of the **Security Content Automation Protocol (SCAP)**, a NIST-maintained suite of specifications for standardised security configuration assessment and vulnerability measurement. Awarded **SCAP 1.2 certification by NIST in 2014** — the only open-source toolkit with this certification.

### Key capabilities
- **Compliance scanning** — audit systems against XCCDF-based benchmarks (CIS, DISA STIG, PCI-DSS, HIPAA, OSPP, ANSSI, CUI)
- **Vulnerability assessment** — evaluate OVAL definitions mapping to CVEs for missing patches
- **Automated remediation** — generate Bash or Ansible fix content from scan results
- **Centralised reporting** — ARF aggregation for dashboards (Satellite, SCAPTAILOR, S3)
- **Policy authoring & tailoring** — custom XCCDF profiles and tailoring files for environment-specific baselines

NIST certification means oscap processes SCAP content per the official SCAP 1.2 specification, making scan results legally admissible for PCI-DSS, SOX, and MAS TRM compliance evidence.

---

## 2. Architecture & SCAP Component Model

### 2.1 SCAP Specifications

| Component | Name | Role |
|-----------|------|------|
| **XCCDF** | Extensible Configuration Checklist Description Format | Defines benchmarks, rules, profiles, scoring |
| **OVAL** | Open Vulnerability and Assessment Language | Machine-readable check logic — probes the live system |
| **CVE** | Common Vulnerabilities and Exposures | Vulnerability IDs mapped in OVAL definitions |
| **CPE** | Common Platform Enumeration | Platform applicability (which OS version a rule covers) |
| **CWE** | Common Weakness Enumeration | Weakness taxonomy for classification |
| **DataStream** | SCAP Source Data Stream Collection | Bundles XCCDF + OVAL + CPE into one portable XML |
| **ARF** | Asset Reporting Format | Self-contained result data stream — complete evidentiary package |

### 2.2 Evaluation Workflow

```
Source DataStream (e.g. ssg-rhel9-ds.xml)
       │
       ▼
oscap xccdf eval --profile <ID>
       │
       ├─► XCCDF Engine iterates rules in the selected profile
       ├─► OVAL Engine evaluates each definition via system probes
       │      (file reads, RPM queries, sysctl, audit subsystem)
       ├─► CPE matching filters non-applicable platforms
       └─► Writes: XCCDF Results XML (rule-level pass/fail + score)
                   ARF DataStream (XCCDF + OVAL + CPE bundled)
                   HTML Report (human-readable tables)
```

### 2.3 Result Formats

| Format | Use Case | Auditor Acceptance |
|--------|----------|-------------------|
| XCCDF Results | CI gating, programmatic consumption | Partial — no OVAL context |
| ARF | Complete evidence, Satellite upload | **Full** — self-contained NIST-certified package |
| HTML Report | Human review, compliance email | Yes — exportable to PDF |
| OVAL Results | Debugging check logic | Internal only |

---

## 3. Installation

**RHEL 8–9 / AlmaLinux / Rocky:** `sudo dnf install -y openscap-scanner scap-security-guide` — optionally `scap-workbench openscap-engine-sce`

**Ubuntu 22.04 / 24.04:** `sudo apt update && sudo apt install -y libopenscap8 scap-security-guide`

**From source:** `git clone https://github.com/OpenSCAP/openscap.git && cd openscap && mkdir build && cd build && cmake .. -DCMAKE_INSTALL_PREFIX=/usr && make -j$(nproc) && sudo make install`

---

## 4. SCAP Security Guide (SSG) Content

The **SCAP Security Guide** (ComplianceAsCode at [GitHub](https://github.com/ComplianceAsCode/content)) provides pre-built SCAP content. Data streams are at `/usr/share/xml/scap/ssg/content/` (RHEL) or `/usr/share/scap-security-guide/` (Ubuntu).

### 4.1 Common Data Streams

`ssg-rhel9-ds.xml` / `ssg-rhel8-ds.xml` — RHEL; `ssg-ubuntu2204-ds.xml` — Ubuntu; `ssg-rhcos4-ds.xml` — OpenShift nodes; `ssg-jre-ds.xml` / `ssg-chromium-ds.xml` — application baselines.

### 4.2 Listing Profiles

```bash
oscap info /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

Profiles include: `cis`, `stig`, `pci_dss`, `hipaa`, `ospp`, `cui`, `e8`, `anssi_*`, `standard`.

---

## 5. Supported Policies & Profiles

| Policy | SSG Profile ID | Domain | Rules (RHEL 9) |
|--------|---------------|--------|----------------|
| CIS Benchmark | `content_profile_cis` | Enterprise hardening | ~700 |
| DISA STIG | `content_profile_stig` | US DoD / defence | ~900 |
| PCI-DSS v3.2.1 | `content_profile_pci_dss` | Payment card industry | ~350 |
| HIPAA | `content_profile_hipaa` | Healthcare | ~250 |
| OSPP | `content_profile_ospp` | Common Criteria / govt | ~400 |
| CUI | `content_profile_cui` | Controlled Unclassified Info | ~450 |
| ACSC E8 | `content_profile_e8` | Australian govt | ~300 |
| ANSSI-BP-028 | `content_profile_anssi_*` | French govt | 3 levels |
| Standard | `content_profile_standard` | General baseline | ~200 |

**Selection guide:** Banking → CIS + PCI-DSS; MAS TRM → CIS tailored; SOX → CIS + OSPP; Defence → DISA STIG; Healthcare → HIPAA; OpenShift → rhcos4 STIG; General → CIS or Standard.

---

## 6. CLI Fundamentals

### 6.1 Command Structure

```
oscap <module> <action> [options] <target>
```

**Core modules:** `info`, `xccdf eval`, `xccdf generate`, `xccdf validate`, `oval eval`, `ds`, `cpe`.

**Key options:** `--profile <ID>` — select profile; `--results <FILE>` / `--results-arf <FILE>` — output; `--report <FILE>` — HTML; `--tailoring-file <FILE>` — customisation; `--fetch-remote-resources` — CVE feeds; `--remediate` — auto-fix; `--rule <ID>` — single rule target.

### 6.2 Inspecting Content

```bash
oscap info /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
oscap info --profile cis /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

---

## 7. Scanning Workflows

### 7.1 Basic Compliance Scan

```bash
sudo oscap xccdf eval \
    --profile xccdf_org.ssgproject.content_profile_cis \
    --results-arf /tmp/arf-results.xml \
    --report /tmp/report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

### 7.2 Vulnerability Scan (CVE)

```bash
# Direct OVAL evaluation
sudo oscap oval eval --results /tmp/oval-results.xml \
    --report /tmp/cve-report.html /tmp/redhat.oval.xml.bz2

# Via data stream with remote fetch
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
    --fetch-remote-resources --report /tmp/report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

### 7.3 Targeted / Single-Rule Scan

```bash
sudo oscap xccdf eval \
    --rule xccdf_org.ssgproject.content_rule_sshd_set_idle_timeout \
    --report /tmp/targeted.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

### 7.4 Scan with Tailoring

```bash
sudo oscap xccdf eval \
    --profile xccdf_org.ssgproject.content_profile_cis \
    --tailoring-file /etc/my-tailoring.xml \
    --results-arf /tmp/custom-arf.xml --report /tmp/custom-report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

### 7.5 Online Remediation (Modifies System)

```bash
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
    --remediate --results-arf /tmp/remediated-arf.xml \
    --report /tmp/remediated-report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

### 7.6 Container & VM Scanning

```bash
oscap-docker image-cve --report /tmp/container-report.html my-registry/my-image:latest
oscap-docker container-cve my-container-name
oscap-vm --report /tmp/vm-report.html --profile cis \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml --vm my-vm-name
```

---

## 8. SCE — Script Check Engine

SCE is an OpenSCAP extension letting XCCDF rules delegate checks to arbitrary shell scripts — useful when OVAL cannot express complex or proprietary checks.

**SCE rule in XCCDF:**
```xml
<Rule id="xccdf_myorg_rule_custom_crypto" selected="true">
  <check system="http://open-scap.org/page/SCE">
    <check-content-ref href="scripts/check_crypto.sh"/>
  </check>
</Rule>
```

**SCE vs OVAL:** OVAL = XML-based, cross-platform, NIST-certified, fast (native probes). SCE = arbitrary scripts, Linux-only, non-standard, slower (process spawn per check). Enable SCE debug: `oscap xccdf eval --check-engine-results /tmp/sce-debug.xml`.

---

## 9. SCAP Workbench (GUI)

```bash
scap-workbench &
```

Features: load SSG data streams, browse profiles with rule drill-down, toggle rules (exports tailoring file), run scans with real-time progress, export HTML/ARF.

**When to use:** Workbench for exploration/ad-hoc scans and interactive tailoring generation. CLI for CI pipelines, bulk production scans, automation.

---

## 10. Reporting

```bash
# Generate HTML from previous results
oscap xccdf generate report --output /tmp/report.html --results-arf /tmp/arf.xml

# Generate hardening guide from a profile
oscap xccdf generate guide \
    --profile xccdf_org.ssgproject.content_profile_cis \
    --output /tmp/cis-guide.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml

# Extract score programmatically
grep -oP 'score="\K[^"]+' /tmp/results.xml | head -1
# Count pass/fail
grep -oP 'result="\K[^"]+' /tmp/results.xml | sort | uniq -c
```

---

## 11. Remediation

```bash
# Generate Bash fix script
oscap xccdf generate fix --fix-type bash --output /tmp/remediation.sh /tmp/results.xml

# Generate Ansible playbook
oscap xccdf generate fix --fix-type ansible --output /tmp/remediation.yml /tmp/results.xml

# Apply
sudo bash /tmp/remediation.sh
ansible-playbook -i inventory.ini /tmp/remediation.yml
```

**Lifecycle:** Scan → Analyse → Generate Fix → Review → Apply → Re-scan → (loop until compliant).

---

## 12. Customization — Tailoring & Custom Profiles

### 12.1 Tailoring Files (Preferred — Upgrade-Safe)

Overrides a base profile without touching original content:

```xml
<xccdf:Tailoring xmlns:xccdf="http://checklists.nist.gov/xccdf/1.2" id="my-tailoring">
  <xccdf:benchmark-reference href="/usr/share/.../ssg-rhel9-ds.xml"/>
  <xccdf:Profile id="custom-cis-tailored">
    <xccdf:title>CIS Tailored for Banking</xccdf:title>
    <xccdf:extends based-on="xccdf_org.ssgproject.content_profile_cis"/>
    <xccdf:select idref="...rule_chronyd_or_ntpd_set_maxpoll" selected="false"/>
    <xccdf:set-value idref="...value_var_accounts_minimum_age_login_defs">14</xccdf:set-value>
  </xccdf:Profile>
</xccdf:Tailoring>
```

### 12.2 Custom XCCDF Profile

For entirely new baselines, author a standalone XCCDF referencing SSG or custom OVAL rules. Package as a data stream:

```bash
oscap ds sds-create --datastream-id my-ds benchmark-xccdf.xml custom-oval.xml > ssg-my-ds.xml
```

---

## 13. Integration Patterns

### 13.1 Red Hat Satellite 6

```bash
hammer scap-content create --title "CIS RHEL9" --scap-file /usr/share/.../ssg-rhel9-ds.xml
hammer compliance-policy create --name "Banking CIS" --scap-content-id 1 --period monthly --weekday sun
```
Results in Monitor → Compliance. ARF files ship to Satellite for centralised view and API access.

### 13.2 OpenShift Compliance Operator

Wraps OpenSCAP for Kubernetes/node scanning:

```yaml
apiVersion: compliance.openshift.io/v1alpha1
kind: ScanSettingBinding
metadata:
  name: banking-compliance
profiles:
  - kind: Profile; name: ocp4-cis-node; apiGroup: compliance.openshift.io/v1alpha1
  - kind: Profile; name: ocp4-cis; apiGroup: compliance.openshift.io/v1alpha1
settingsRef:
  kind: ScanSetting; name: periodic-daily
```
```bash
oc create -f banking-compliance.yaml; oc get compliancescan -w
oc patch compliancecheckresult <check> -p '{"spec":{"status":"RemediationApplied"}}' --type merge
```

### 13.3 CI/CD Pipeline Gating

```yaml
openscap-scan:
  stage: security
  script:
    - dnf install -y openscap-scanner scap-security-guide
    - oscap xccdf eval --profile cis --results-arf arf.xml --report report.html /usr/share/.../ssg-rhel9-ds.xml
    - SCORE=$(grep -oP 'score="\K[^"]+' arf.xml | head -1)
    - if (( $(echo "$SCORE < 80.0" | bc -l) )); then echo "FAIL: score $SCORE below 80%"; exit 1; fi
  artifacts:
    paths: [report.html, arf.xml]
```

### 13.4 Ansible Automation

```yaml
---
- hosts: all; become: yes
  tasks:
    - package: name=openscap-scanner state=present
    - command: oscap xccdf eval --profile cis --results-arf /tmp/arf.xml
      register: scan; ignore_errors: yes
    - command: oscap xccdf generate fix --fix-type ansible --output /tmp/fix.yml /tmp/arf.xml
      when: scan.rc == 0
    # Apply with ansible-playbook or include_role
```

### 13.5 Centralised Reporting (No Satellite)

```bash
#!/bin/bash
# Run post-scan on each target
oscap xccdf eval --profile cis --results-arf "/tmp/arf-$(hostname)-$(date +%Y%m%d).xml"
aws s3 cp "/tmp/arf-$(hostname)-*.xml" "s3://compliance-reports/$(date +%Y/%m/%d)/"
curl -X POST https://compliance.internal.cacib/reports/upload -F "file=@/tmp/arf-*.xml"
```

---

## 14. Banking Use Cases

### 14.1 MAS TRM Alignment (Monetary Authority of Singapore)

For Crédit Agricole CIB Singapore:

| MAS TRM Control | Implementation |
|----------------|----------------|
| TRM 6.1.1 — Config baseline | CIS scan; tailoring for MAS overrides |
| TRM 6.1.2 — Secure config | `oscap xccdf eval` vs tailored CIS (quarterly) |
| TRM 6.1.4 — Vulnerability mgmt | `oscap oval eval` against latest CVE feeds |
| TRM 6.1.5 — Patch mgmt | OVAL CVE scanning pre/post patch windows |
| TRM 6.2 — Hardening baseline | Custom XCCDF extends SSG with MAS-specific rules |
| TRM 7.1 — Audit logging | `audit_rules_*`, `partition_for_/var/log_audit`, `rsyslog_remote_loghost` |

**Quarterly workflow:** Scan (CIS + MAS Tailoring) → ARF + HTML → submit to Compliance → generate Ansible fix → remediate → re-scan to verify.

### 14.2 SOX Compliance Evidence

```bash
oscap xccdf eval \
  --rule xccdf_org.ssgproject.content_rule_audit_access_success \
  --rule xccdf_org.ssgproject.content_rule_accounts_passwords_pam_faillock_deny \
  --rule xccdf_org.ssgproject.content_rule_file_permissions_etc_shadow \
  --report /tmp/sox-report.html \
  /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

**Audit evidence package:** (1) ARF file — complete machine-readable evidence, (2) HTML report — pass/fail per control, (3) remediation script + execution log, (4) re-scan report showing closure.

### 14.3 PCI-DSS Evidence Gathering

| Req | SSG Rules | Mechanism |
|-----|-----------|-----------|
| Req 2 — Change defaults | `sshd_config`, `file_owner_grub2_cfg` | OVAL file probe |
| Req 4 — Encrypt data | `sshd_use_approved_ciphers`, `sysctl_crypto_fips_enabled` | OVAL + sysctl |
| Req 7 — Restrict access | `file_permissions_*` | OVAL file probe |
| Req 8 — Strong auth | `accounts_password_*`, `sshd_set_max_auth_tries` | OVAL + config |
| Req 10 — Audit trails | `audit_rules_*` | OVAL audit probe |
| Req 11 — Regular testing | Monthly `oscap eval` | Cron / pipeline |

```bash
oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_pci_dss \
    --results-arf "/tmp/pcidss-$(hostname)-$(date +%Y%m%d).xml" \
    --report "/tmp/pcidss-$(hostname)-$(date +%Y%m%d).html" \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

### 14.4 Automated Hardening Baseline Enforcement

```bash
#!/bin/bash
# /usr/local/bin/enforce-banking-baseline.sh — cron job
set -euo pipefail
PROFILE="xccdf_org.ssgproject.content_profile_cis"
SSG_DS="/usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml"
TAILORING="/etc/security/cacib-cis-tailoring.xml"
ARF="/tmp/arf-$(hostname)-$(date +%Y%m%d-%H%M).xml"
RPT="/var/www/html/compliance/$(hostname)-$(date +%Y%m%d).html"

oscap xccdf eval --profile "$PROFILE" --tailoring-file "$TAILORING" \
    --results-arf "$ARF" "$SSG_DS"
oscap xccdf generate report --output "$RPT" --results-arf "$ARF"
oscap xccdf generate fix --fix-type ansible \
    --output "/tmp/fix-$(hostname)-$(date +%Y%m%d).yml" "$ARF"
[[ -f /etc/security/auto-remediate.flag ]] && \
    ansible-playbook --limit "$(hostname -f)" "/tmp/fix-$(hostname)-$(date +%Y%m%d).yml"
oscap xccdf eval --profile "$PROFILE" --tailoring-file "$TAILORING" \
    --results-arf /tmp/postfix-arf.xml \
    --report "${RPT%.html}-postfix.html" "$SSG_DS"
```

---

## 15. Troubleshooting & Best Practices

### Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| No profiles | Wrong SSG path / missing package | Reinstall `scap-security-guide` |
| OVAL probe unsupported | Not compiled for platform | Check `oscap --version` |
| Remote resources skipped | Missing `--fetch-remote-resources` | Add flag or pre-download OVAL |
| Low score on prod | Profile too strict | Tailor out conflicting rules |
| Remediation errors | Pre-requisite not met | Apply fixes singly |
| Permission denied | Non-root scanner | Always use `sudo` |

### Best Practices

1. **Always scan as root** — OVAL probes need `/etc/shadow`, kernel params, audit logs.
2. **Tailor, don't fork** — Use tailoring files over modifying SSG. Stay upgrade-safe.
3. **Automate scanning** — systemd timer or cron weekly; ship ARF to central collector.
4. **Banking: combine CIS + PCI-DSS** — CIS for general hardening, PCI-DSS for card data. Use a tailoring file extending `cis` with `pci_dss` rules enabled.
5. **Re-scan after remediation** — Verify fixes resolved findings.
6. **Archive ARF** — Self-contained evidence. Store in immutable storage (S3 Object Lock).
7. **Test remediations in staging** — Some rules may disrupt production services.
8. **Pin SSG versions in CI** — Deterministic results across builds.
9. **Gate CI on score** — Fail if score < threshold (e.g., 80%).
10. **Integrate with ticketing** — Parse ARF results; auto-create tickets for failed rules.

### Useful Aliases

```bash
alias oscap-cis='sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
  --results-arf /tmp/cis-arf.xml --report /tmp/cis-report.html \
  /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml'
alias oscap-fix='oscap xccdf generate fix --fix-type bash --output /tmp/fix.sh'
alias oscap-report='oscap xccdf generate report --output /tmp/report.html --results-arf'
alias oscap-guide='oscap xccdf generate guide --output /tmp/guide.html'
```

---

## References

| Resource | URL |
|----------|-----|
| OpenSCAP Portal | https://www.open-scap.org/ |
| GitHub — OpenSCAP | https://github.com/OpenSCAP/openscap |
| SSG / ComplianceAsCode | https://github.com/ComplianceAsCode/content |
| OpenSCAP User Manual | https://static.open-scap.org/openscap-1.3/oscap_user_manual.html |
| RHEL 9 Security Hardening | https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/ |
| OpenShift Compliance Operator | https://docs.openshift.com/container-platform/latest/security/compliance_operator/ |
| NIST SP 800-126 Rev 3 | https://csrc.nist.gov/publications/detail/sp/800-126/rev-3/final |
| MAS TRM Guidelines | https://www.mas.gov.sg/regulation/guidelines/technology-risk-management-guidelines |
| CIS Benchmarks | https://www.cisecurity.org/cis-benchmarks/ |

---

*Document version 1.0 — July 2026. Maintained by Jack Liu Shurui, Crédit Agricole CIB Singapore.*
