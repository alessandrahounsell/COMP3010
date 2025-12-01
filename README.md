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
## Installation & Data Preparation
## Conclusion
## References 

