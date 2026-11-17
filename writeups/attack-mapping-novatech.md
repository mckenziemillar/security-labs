# Mapping a Breach to MITRE ATT&CK — The NovaTech Scenario

**Based on:** TryHackMe "Cyber Kill Chain" room + a breach scenario analysis

## Goal
Take a data-breach scenario and map each attacker action to a specific MITRE
ATT&CK tactic and technique, to practice turning a messy sequence of events into
structured, defensible analysis.

## Tools / frameworks
- [MITRE ATT&CK](https://attack.mitre.org/) and the ATT&CK Navigator
- Lockheed Martin Cyber Kill Chain

## The scenario
NovaTech, a mid-sized HR-software company, noticed unusual outbound traffic from
an internal HR server. A forensic review reconstructed the attack:

1. Scanned the public web app and found a misconfigured API endpoint
2. Exploited it with a crafted request to reach the internal network
3. Enumerated user accounts and found a service account with weak credentials
4. Used that account to move to a file server with employee records
5. Disabled logging on the compromised server
6. Compressed the data and exfiltrated it over DNS tunneling
7. Used scheduled tasks to persist for several days before detection

## ATT&CK mapping

| Step | Technique | ID | Tactic |
|---|---|---|---|
| Scan web app | Active Scanning | T1595 | Reconnaissance |
| Exploit API | Exploit Public-Facing Application | T1190 | Initial Access |
| Enumerate accounts | Account Discovery | T1087 | Discovery |
| Reach file server | Remote Services | T1021 | Lateral Movement |
| Reuse service account | Valid Accounts | T1078 | Defense Evasion / Persistence |
| Disable logging | Impair Defenses: Disable or Modify Tools | T1562.001 | Defense Evasion |
| Compress data | Archive Collected Data | T1560 | Collection |
| DNS tunneling | Exfiltration Over Alternative Protocol | T1048.003 | Exfiltration |
| Scheduled tasks | Scheduled Task | T1053.005 | Persistence |

**Note on technique choice:** I mapped the move to the file server as **Remote
Services (T1021)** for the lateral-movement step and kept **Valid Accounts
(T1078)** as the reason it went unnoticed — using real credentials let the
attacker blend in, which is defense evasion and persistence rather than a
movement technique on its own.

## What the mapping shows
The breach didn't rely on exotic tooling. It chained ordinary things — valid
credentials, scheduled tasks, DNS traffic — into a quiet, patient campaign.
ATT&CK turned the event into categories I could reason about: disabling logging
reads immediately as Defense Evasion, and the map exposes exactly where NovaTech
failed — no DNS monitoring, over-permissioned service accounts, and logging that
could simply be switched off.

## Defender takeaways
- Monitor DNS volume and destinations; treat it as an exfil channel, not just
  background traffic
- Least-privilege service accounts, strong credentials, and alerting on their use
- Protect logging integrity (forward logs off-host so they can't be wiped locally)
