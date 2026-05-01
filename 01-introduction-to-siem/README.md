# TryHackMe: Introduction to SIEM

**Room:** [Introduction to SIEM](https://tryhackme.com/room/introtosiem)
**Path:** Core SOC Solutions
**Difficulty:** Easy
**Time:** 120 minutes
**Status:** Completed (100%)

---

## Overview

This room introduces Security Information and Event Management (SIEM) as the central solution a SOC analyst uses to monitor, detect, and investigate security events across an enterprise environment. The room covers why isolated logs are insufficient for modern security operations, how SIEM platforms solve the visibility problem through centralised collection and normalisation, and how detection rules translate raw events into actionable alerts.

## Skills Demonstrated

- Identifying host-centric and network-centric log sources
- Reading and interpreting Windows, Linux, and Apache logs
- Understanding SIEM ingestion methods (agent, syslog, manual upload, port forwarding)
- Reviewing detection rule logic and matching conditions to events
- Triaging an alert end-to-end: identifying the responsible process, user, and host
- Classifying alerts as true positive or false positive
- Selecting an appropriate response action

## Key Concepts

### Log Sources: Host-Centric vs Network-Centric

A SOC analyst works with two broad categories of telemetry. Recognising which is which guides where to look during an investigation.

**Host-centric logs** capture activity occurring on or about an endpoint:
- User authentication attempts
- File access events
- Process execution
- Registry modifications
- PowerShell execution

**Network-centric logs** capture communication between hosts or with external resources:
- VPN connections
- SSH sessions
- Web traffic (proxy, web server)
- FTP file transfers
- Network file shares

The same incident usually leaves traces in both categories. A compromised endpoint will show host-centric signs (unusual process execution) and network-centric signs (outbound connection to a suspicious IP).

### Why Isolated Logs Fail

Before SIEM, analysts faced four blocking problems:

1. **Volume:** Hundreds of events per second across dozens of devices
2. **No centralisation:** Logs sit on the device that produced them, requiring SSH or RDP into each one
3. **Limited context:** A single log entry rarely tells the full story
4. **Format inconsistency:** Windows, Linux, firewalls, and applications all produce logs in different shapes

The example given in the room makes the context point well. A user accessing a file is normal. The same file accessed by a user who only just authenticated via VPN from an unfamiliar IP after lateral movement from another compromised host is an incident. You only see that pattern when logs are correlated across sources.

### What SIEM Provides

A SIEM solution addresses each of those gaps:

- **Centralised log collection** through lightweight agents (Splunk calls them forwarders), syslog, or APIs
- **Parsing and normalisation** so a Windows authentication event and a Linux auth event can be queried with the same field names
- **Correlation** across log sources to surface multi-stage activity
- **Real-time alerting** based on detection rules
- **Dashboards and reporting** for at-a-glance situational awareness

### Log Ingestion Methods

Four common ways logs reach a SIEM:

| Method | How it works | Typical use |
|--------|--------------|-------------|
| Agent / Forwarder | Lightweight tool installed on the endpoint pushes logs to the SIEM | Windows and Linux endpoints, servers |
| Syslog | Standard protocol streaming logs to a central destination | Network devices, web servers, databases |
| Manual upload | Offline log files ingested for analysis | Forensic investigation, retrospective analysis |
| Port forwarding | SIEM listens on a port; endpoints push data to it | Custom integrations, appliances |

### Detection Rules in Practice

Detection rules are logical expressions evaluated against incoming events. The room walks through two practical examples that map directly to real SOC work:

**Use case 1: Detecting log clearing**
Adversaries clear event logs to cover their tracks. Windows logs Event ID 104 every time logs are cleared.
> If `LogSource = WinEventLog` AND `EventID = 104` then alert: `Event Log Cleared`

**Use case 2: Detecting reconnaissance commands**
Post-exploitation, attackers commonly run `whoami` to confirm context. Process execution is captured by Event ID 4688.
> If `LogSource = WinEventLog` AND `EventCode = 4688` AND `NewProcessName contains whoami` then alert: `WHOAMI command executed`

These rules are simple, but the principle is the foundation of detection engineering: identify a behaviour worth watching, find the field-value combination that captures it, and write the logic.

### Alert Investigation Workflow

When an alert fires, the analyst follows a consistent triage flow:

1. Open the alert and review the events that triggered it
2. Cross-check against the detection rule conditions
3. Determine true positive or false positive
4. Take action:
   - **False positive:** Tune the rule to reduce noise
   - **True positive:** Escalate, contact the asset owner, isolate the host, block the source IP

## Lab Walkthrough

The lab in Task 6 simulates a SIEM dashboard. Triggering a "suspicious activity" event causes a rule to fire, and the analyst works through the standard triage flow.

**What I did:**

1. Started the suspicious activity from the dashboard
2. Identified the alert that fired and traced it back to the source process
3. Examined the event details to identify the user responsible and the hostname involved
4. Reviewed the detection rule and matched the term in the event that satisfied the rule condition
5. Classified the event (true positive vs false positive) based on the rule logic and event context
6. Selected the correct response action, which revealed the room flag

This mirrors the exact sequence a Tier 1 SOC analyst follows on every alert: triage, validate, classify, respond.

## How This Maps to a SOC Analyst Role

| Lab activity | SOC analyst equivalent |
|--------------|----------------------|
| Reviewing the triggered alert | Picking up a ticket from the queue |
| Identifying the process, user, and host | Establishing the "who, what, where" of the event |
| Matching event fields to rule conditions | Validating the alert against the detection logic |
| True positive / false positive call | Classifying the alert in the SIEM and ticketing system |
| Selecting the response action | Following the playbook: contain, escalate, or close |

Every step in the lab is a step in the standard SOC alert lifecycle. The platform changes (Splunk, Sentinel, QRadar, Elastic) but the workflow does not.

## Key Takeaways

- SIEM exists because the volume, distribution, and format diversity of logs makes manual investigation impossible at scale
- Normalisation is what makes correlation possible. Without it, you cannot write a rule that works across log sources
- Detection rules are deliberately simple. Quality comes from selecting the right field-value combinations, not from complexity
- The alert-to-action workflow (triage, validate, classify, respond) is the same regardless of platform. Learning it once transfers everywhere
- False positives are not failures. Tuning is a core part of detection engineering, and a rule that never produces false positives is probably missing real ones

## Tools and Concepts Referenced

- Splunk (SIEM platform)
- Windows Event Viewer and Event IDs (104, 4688)
- Linux log paths (`/var/log/auth.log`, `/var/log/secure`, `/var/log/httpd`, `/var/log/cron`, `/var/log/kern`)
- Apache access log format
- Syslog protocol
- Detection rule syntax (logical conditions on field-value pairs)

## Next Steps

This room sits at the start of TryHackMe's Core SOC Solutions path. From here I am moving on to:

- **Splunk: The Basics** - hands-on with SPL queries
- **Investigating with Splunk** - applied incident investigation
- **Investigating with ELK** - the same workflow on the Elastic Stack
- **Incident Handling with Splunk** - end-to-end IR practice

---

*Room completed as part of the TryHackMe SOC Level 1 learning path. Writeup focuses on concepts and methodology rather than reproducing room answers.*
