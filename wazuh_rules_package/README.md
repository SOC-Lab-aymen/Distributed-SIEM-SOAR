# SIG Wazuh Rules Package

This directory contains the finalized phase-based Wazuh custom rule files. Each XML file represents a specific detection phase and must be validated against the telemetry available in the Wazuh deployment before it is enabled.

> These files define detection logic. They do not automatically create the required agent log collection, FIM paths, decoders, database audit settings, Docker event source, or external integrations.

## Rule files

| Phase | File | Scope | Status |
|---|---|---|---|
| Phase 1 | `phase_01_linux_auth_admin.xml` | Linux and SSH authentication and administration | Initial baseline |
| Phase 1B | `phase_01b_windows_onprem.xml` | Windows authentication and endpoint activity | Separate on-premises phase |
| Phase 2 | `phase_02_fim_configuration.xml` | File integrity and sensitive configuration monitoring | Requires explicit FIM paths |
| Phase 3 | `phase_03_web_postgresql_templates.xml` | Web, Traefik, and PostgreSQL detections | Requires real decoded web/database events |
| Phase 4 | `phase_04_docker_templates.xml` | Docker lifecycle, privilege, image, and host-port detections | Requires Docker or host-inventory telemetry |
| Phase 5 | `phase_05_network_suricata_templates.xml` | Suricata and network detections | Deferred until Suricata is deployed |
| Phase 6 | `phase_06_operations_soar_templates.xml` | Vulnerability, SCA, operations, and SOAR candidates | Apply selectively after prerequisites exist |
| Phase 7 | `phase_07_gitlab_cicd.xml` | GitLab and CI/CD integrity | Later phase |

## Loading order

Load the files in the following order, validating each phase before moving to the next:

```text
Phase 1 → Phase 1B → Phase 2 → Phase 3 → Phase 4 → Phase 6 → Phase 7
```

Phase 5 is intentionally excluded from the current loading sequence because its rules require a deployed Suricata sensor and collected `eve.json` events. Phase 7 should be loaded only when GitLab audit and repository telemetry are available.

Do not load the original and finalized versions of the same rule files simultaneously. They may define overlapping custom IDs and can produce duplicate or conflicting alerts.

## Phase 1 — Linux authentication and administration

File: `phase_01_linux_auth_admin.xml`

This phase contains Linux and SSH rules for invalid-user attempts, SSH brute-force activity, root login, failed sudo authentication, local-user creation or deletion, and root crontab modification.

| Rule IDs | Coverage |
|---|---|
| `100200` | SSH login using a nonexistent user |
| `100201` | SSH brute force from one source IP |
| `100202` | Successful SSH root login |
| `100204` | Failed sudo authentication |
| `100205` | New Linux local user |
| `100206` | Linux local user or group deletion |
| `100207` | Root crontab modification |

Before loading this file, confirm that the Linux agent is connected and that `/var/log/auth.log` or the equivalent authentication source is arriving at the manager. Test representative SSH and sudo events with `/var/ossec/bin/wazuh-logtest`.

Rule `100203` is intentionally not present in the finalized package.

## Phase 1B — Windows on-premises endpoint rules

File: `phase_01b_windows_onprem.xml`

This phase is kept separate from the Linux baseline and contains Windows failed-logon, brute-force, account lockout, account creation, privileged-group, service, scheduled-task, PowerShell, audit-log, Defender, and privileged-logon rules.

| Rule IDs | Coverage |
|---|---|
| `100300–100304` | Failed logon, brute force, lockout, account creation, and group changes |
| `100305–100306` | Service installation and scheduled-task changes |
| `100307` | PowerShell encoded command |
| `100308–100309` | Security audit-log clearing and Defender detection |
| `100310` | Privileged interactive or remote-interactive logon |

Before loading this file, confirm that Windows Security events are collected. Enable and verify the PowerShell Operational, Defender, Task Scheduler, System, and Sysmon sources only when the corresponding rules are required.

Validate `100300` with a controlled failed logon. Validate `100301` only with an approved disposable test account and the documented five-failure/60-second threshold. A failed-logon test does not by itself prove RDP-specific detection.

## Phase 2 — File integrity and sensitive configuration

File: `phase_02_fim_configuration.xml`

This phase contains FIM rules for critical files, web roots, SSH configuration, cron/systemd persistence, Windows paths and registry autoruns, backup/artifact paths, and sensitive service configuration.

| Rule IDs | Coverage |
|---|---|
| `100400–100407` | Critical Linux/Windows files, web roots, SSH, persistence, permissions, and registry autoruns |
| `100411` | Backup or artifact modification |
| `100412` | `pg_hba.conf` or `postgresql.conf` modification |
| `100413` | Traefik dynamic configuration modification |
| `100414` | `docker-compose.yml` or `.env` modification |
| `100415` | `/etc/passwd`, `/etc/shadow`, or `/etc/sudoers` modification |
| `100416` | SSH host keys or `authorized_keys` modification |

Before loading this file, configure the relevant paths in the agent’s `<syscheck>` section. FIM rules do not monitor files merely because a rule references them.[1]

The intended sensitive-file scope includes:

```text
pg_hba.conf
postgresql.conf
Traefik dynamic configuration
docker-compose.yml
.env
/etc/passwd
/etc/shadow
/etc/sudoers
SSH host keys
authorized_keys
```

Use narrow paths and treat `.env`, `/etc/shadow`, private keys, and similar files as sensitive. Test each path with a harmless reversible modification and confirm that the alert identifies the expected file.

GitLab rules are intentionally not included in this phase. They are located in Phase 7.

## Phase 3 — Web and PostgreSQL rules

File: `phase_03_web_postgresql_templates.xml`

This phase contains web-request patterns, web brute force, directory traversal, XSS indicators, PostgreSQL errors and authentication failures, destructive-query indicators, and PostgreSQL administration rules.

| Rule IDs | Coverage |
|---|---|
| `100500` | Suspicious SQL-injection request pattern |
| `100501` | Web-to-PostgreSQL correlation template |
| `100502–100504` | PostgreSQL errors, authentication failures, and destructive-query indicators |
| `100505–100507` | Web brute force, directory traversal, and XSS patterns |
| `100508–100512` | PostgreSQL roles, privileges, extensions, configuration, and privileged functions |

Before loading this file, collect and decode the real Traefik/application and PostgreSQL events. Confirm the decoder names and actual field paths with `wazuh-logtest`.

Rule `100501` is a correlation template, not an independent confirmation of SQL injection. Rule `100500` identifies a suspicious web request. A related PostgreSQL error or audit event must then share a demonstrated correlation key—such as source IP, request ID, or database user—within a documented time window. If no reliable shared key exists, retain separate indicative alerts and do not describe the result as confirmed SQL injection.

Use disposable applications and databases for validation. Do not execute destructive statements against production databases.

## Phase 4 — Docker and host-level port exposure

File: `phase_04_docker_templates.xml`

This phase contains Docker container creation, privileged configuration, lifecycle activity, `docker exec`, image pulls, image findings, and host-level port exposure.

| Rule IDs | Coverage |
|---|---|
| `100600` | Container creation |
| `100601` | Privileged container configuration |
| `100602` | Container lifecycle event |
| `100603` | Command execution inside a container |
| `100604` | Image pull requiring registry validation |
| `100605` | Host-level port exposure by a container |
| `100606` | High-risk or unsupported image finding |

Before loading this file, confirm that Docker events or reliable host/container inventory data are reaching the Wazuh manager. Docker Desktop running by itself does not prove that Wazuh is receiving the required events.

Rule `100605` is intended to compare container-published host ports and host listening-port inventory against an approved exposure list. It does not detect or authorize public Docker API exposure. Never expose the Docker API publicly as a test.

Validate with disposable containers and remove them after testing.

## Phase 5 — Suricata and network rules

File: `phase_05_network_suricata_templates.xml`

This phase contains future rules for Suricata alerts, network scans, command-and-control or malware network activity, threat-intelligence matches, and anomalous DNS patterns.

| Rule IDs | Coverage |
|---|---|
| `100700–100704` | Suricata and network detections |

This phase is **postponed**. Do not load it until an approved Suricata sensor is deployed, `eve.json` is generated, the file is collected by Wazuh, and representative events are decoded. Wazuh supports Suricata ingestion, but the sensor and telemetry must exist before these rules can be validated.[2]

## Phase 6 — Operations and SOAR candidates

File: `phase_06_operations_soar_templates.xml`

This phase contains local vulnerability, SCA, listening-port, disk, memory, service, and time-change detections, together with SOAR candidate rules.

| Rule IDs | Coverage |
|---|---|
| `100800–100802` | Vulnerability, SCA, and unexpected service/port findings |
| `100804–100807` | Disk, memory, service, and clock operational events |
| `100900–100902` | SOAR candidate, human approval, and enrichment events |

Before loading this file, confirm that the relevant Wazuh modules and system logs are available. Use synthetic resource thresholds rather than exhausting production storage or memory. Agent or manager availability should also be documented through platform monitoring.

Rules `100808`, `100809`, `100810`, and `100811` were removed from the finalized package. Cloud-provider API and audit monitoring is outside the current rule-file scope.

SOAR candidate rules must not be described as automatic containment. Any response workflow requires tested notification, enrichment, approval, case creation, permissions, and audit evidence.

## Phase 7 — GitLab and CI/CD rules

File: `phase_07_gitlab_cicd.xml`

This later phase contains GitLab configuration, pipeline, runner-token/configuration, and protected-branch or repository-permission monitoring.

| Rule IDs | Coverage |
|---|---|
| `100408–100410` | GitLab configuration, CI/CD pipeline, and runner changes |
| `100417` | Protected branch, membership, or repository permission change |

Before loading this file, confirm that GitLab audit events, repository events, runner events, and relevant configuration paths are being collected. Validate with a disposable test project and a reversible change.

## Validation procedure

For every rule file, follow this sequence:

1. Confirm the required telemetry source is active.
2. Back up the currently loaded rules.
3. Copy only the selected phase file into the manager’s mounted rules directory.
4. Test representative raw events with `/var/ossec/bin/wazuh-logtest`.
5. Confirm the decoder, fields, parent rule, custom rule ID, severity, and description.
6. Reload the manager only after testing succeeds.
7. Generate a safe controlled event in a disposable lab asset.
8. Search the dashboard by rule ID and agent name.
9. Record the alert, timestamp, test action, and screenshot.
10. Roll back the test change and tune the rule if necessary.

## Rule-management requirements

Custom rules should use the project’s assigned ID ranges and should be stored in separate files by phase or rule family. Avoid duplicate custom IDs and avoid duplicating built-in Wazuh detections unless the custom rule has a documented difference in threshold, severity, asset scope, or response purpose.[3]

When a decoder field or event structure differs from the assumptions in a template, modify the condition only after capturing a real raw event and confirming its decoded fields. Do not claim that a template is operational until its event source and validation evidence exist.

## Exclusions applied

The finalized files apply the following catalogue decisions:

| Item | Final treatment |
|---|---|
| `100203` | Removed |
| `103301` | Not included |
| GitLab rules | Moved from Phase 2 to Phase 7 |
| PostgreSQL administration rules | Added to Phase 3 |
| `100605` | Changed to host-level container port exposure |
| Suricata | Postponed to a later stage |
| `100808–100811` | Removed |
| Sensitive configuration files | Added to Phase 2 FIM scope |

## References

[1] [Wazuh, File integrity monitoring](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/index.html)

[2] [Wazuh, Network IDS integration with Suricata](https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html)

[3] [Wazuh, Custom rules](https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html)
