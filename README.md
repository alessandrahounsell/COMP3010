# Frothly BOTSV3 Incident Analysis

## Introduction

This report documents the simulated security incident within the fictional brewing company named “Frothly”. In this writeup I will be analysing the collections of logs provided in the publicly available dataset: BOTSV3 to investigate the attack following the cyber kill chain methodology.

#### Objectives:
<ol>
<li> Identify the root cause and initial vector of the compromise. </li>
<li> Figure out the scope of the compromise, is the whole network affected or just specific machines. </li>
<li> Track the attack's progression through the system identifying the tactics, techniques and procedures used (TTPs). </li>
<li> Determine the impact including if any sensitive customer or company data has been successfully exfiltrated from the system. </li>
</ol>
<br>

### Scope:

The report below is inclusive of the data provided by the BOTSV3 dataset, available here: https://github.com/splunk/botsv3 , within Splunk. Including network, endpoint, email, and cloud service data from environments like Amazon AWS and Microsoft Azure. This is a post-incident analysis and as such excludes external information gathering such as the implementation of any immediate attack mitigations and any direct interaction with infected machines or networks.

### Assumptions:

<ul>
<li> Data integrity: The logs are complete and correct, no data has been altered or tampered with in any way. </li>
<li> Time Synchronicity: All events/ logs within the data set are correctly indexed and time synchronised. </li>
</ul>
<br>


## SOC Roles & Incident Handling Reflection 

A SOC team is ‘a centralised team responsible for improving an organisation’s cybersecurity posture by preventing, detecting and responding to threats.’ [1]  A SOC consists of a combination of cybersecurity professionals, including three tiers of cybersecurity analysts whos roles are as follows:
<ul>
<li> <b> Tier 1 - Triage: </b> Initial monitoring and prioritising of alerts. Gathering preliminary evidence to enrich alert data with context by correlating events from multiple platforms, [2] which can then be escalated to tier 2 analysts. [3] </li>
<li> <b> Tier 2 -  Incident Responder: </b> Core in-depth investigation of alerts escalated by the tier 1 analyst. Identifies the affected systems and scope of the attack and conducts comprehensive log analysis and forensic examination. Implementing appropriate containment/ remediation strategies. </li>
<li> <b> Tier 3  - Threat Hunter: </b> Takes a proactive approach searching for suspicious activity before it becomes an incident and leading network testing such as vulnerability and PEN testing. Also serves as an escalation point for tier 2 analysts analysing complex incidents. </li>
</ul>

## Installation & Data Preparation
## Conclusion
## References 

[1] https://www.microsoft.com/en-us/security/business/security-101/what-is-a-security-operations-center-soc <br>
[2] https://radiantsecurity.ai/learn/soc-tier-1-vs-tier-2-vs-tier-3/  <br>
[3] https://www.crowdstrike.com/en-us/cybersecurity-101/next-gen-siem/security-operations-center-soc/ 


