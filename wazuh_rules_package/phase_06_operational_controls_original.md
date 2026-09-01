# Phase 6 Operational Controls Outside Custom XML

## Purpose

Not every important SOC detection is a Wazuh custom rule. The following controls must be configured in the Wazuh platform, operating system, dashboard, or notification layer. They complement the Phase 6 XML file and complete the real-lab detection catalogue.

| Control ID | Control | Why it is not a standalone custom XML rule | Required SIG evidence |
|---:|---|---|---|
| 100803 | Wazuh agent, manager, and log-collection unavailability | A disconnected agent or unavailable manager is a platform health state, not necessarily an incoming event with a safely matchable decoder. | Agent status dashboard/scheduled health query, notification threshold, test by stopping a non-production agent, and recovery record. |

## Required configuration approach

Configure a dashboard view and an alerting/notification process for any agent that becomes disconnected or has not reported within the approved service window. Monitor the Wazuh manager, indexer, dashboard, log collector, disk capacity, and agent count through the SIG operations process. Test the alert with an approved non-production agent only.

The validated result should state the detection threshold, notification recipient, escalation timing, affected agent, recovery time, and the person responsible for triage.
