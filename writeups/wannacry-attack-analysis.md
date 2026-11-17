# WannaCry Ransomware — Case Study Analysis

**Based on:** MITRE ATT&CK case-study lab

## Goal
Analyze a real-world breach, identify the ATT&CK techniques behind it, and
explain how the attack phases connected.

## Summary
WannaCry (May 2017) exploited **EternalBlue**, a vulnerability in Microsoft's
SMBv1 protocol originally developed by the NSA and later leaked. Once on a
system, it encrypted the user's files and demanded $300–$600 in Bitcoin for the
decryption key. Within hours it reached 230,000+ computers across 150 countries,
hitting the UK NHS, FedEx, Renault, and Telefónica. The global spread stopped
when a researcher registered a domain found in the code, triggering a built-in
kill switch.

## ATT&CK techniques

| Technique | ID | Tactic |
|---|---|---|
| Exploitation of Remote Services | T1210 | Lateral Movement |
| Remote Services: SMB/Windows Admin Shares | T1021.002 | Lateral Movement |
| Data Encrypted for Impact | T1486 | Impact |
| System Services: Service Execution | T1569.002 | Execution |

**Corrections from my first pass:** the SMB sub-technique is **T1021.002** (I'd
written a non-existent ID). EternalBlue/SMB exploitation is **Lateral Movement**,
which is how the worm spread internally, rather than Initial Access. I also
swapped PowerShell (T1059.001) for **Service Execution (T1569.002)**, since
WannaCry ran via a Windows service rather than PowerShell scripting.

## Analysis
- **Attacker:** attributed to the Lazarus Group (North Korea).
- **Victims:** governments, hospitals, and corporations across 150 countries,
  plus individual users.
- **Desired outcome:** financial gain through Bitcoin ransom, with large-scale
  disruption as a likely secondary effect.

## How the phases connected
SMB exploitation gave access and let the worm spread **laterally** on its own —
no phishing or user interaction needed. On each host it **executed** via a
service, then **encrypted** files to force payment. The self-propagation is what
turned a single exploit into a global incident in hours.
