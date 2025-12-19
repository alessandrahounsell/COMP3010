# Frothly BOTSV3 Incident Analysis

## Introduction

This report documents the simulated security incident within the fictional brewing company named “Frothly”. In this writeup I will be analysing the collections of logs provided in the publicly available dataset: BOTSV3 to investigate the attack following the cyber kill chain methodology.

### Objectives:
<ol>
<li> Identify the root cause and initial vector of the compromise. </li>
<li> Figure out the scope of the compromise, is the whole network affected or just specific machines. </li>
<li> Track the attack's progression through the system identifying the tactics, techniques and procedures used (TTPs). </li>
<li> Determine the impact including if any sensitive customer or company data has been successfully exfiltrated from the system. </li>
</ol>

### Scope:

The report below is inclusive of the data provided by the BOTSV3 dataset, available here: https://github.com/splunk/botsv3 , within Splunk. Including network, endpoint, email, and cloud service data from environments like Amazon AWS and Microsoft Azure. This is a post-incident analysis and as such excludes external information gathering such as the implementation of any immediate attack mitigations and any direct interaction with infected machines or networks.

### Assumptions:

<ul>
<li> Data integrity: The logs are complete and correct, no data has been altered or tampered with in any way. </li>
<li> Time Synchronicity: All events/ logs within the data set are correctly indexed and time synchronised. </li>
</ul>
<br>


## SOC Roles & Incident Handling Reflection 

### SOC Roles

A SOC team is ‘a centralised team responsible for improving an organisation’s cybersecurity posture by preventing, detecting and responding to threats.’ [1]  A SOC consists of a combination of cybersecurity professionals, including three tiers of cybersecurity analysts whose roles are as follows:
<ul>
<li> <b> Tier 1 - Triage: </b> Initial monitoring and prioritising of alerts. Gathering preliminary evidence to enrich alert data with context by correlating events from multiple platforms, [2] which can then be escalated to tier 2 analysts. [3] </li>
<li> <b> Tier 2 -  Incident Responder: </b> Core in-depth investigation of alerts escalated by the tier 1 analyst. Identifies the affected systems and scope of the attack and conducts comprehensive log analysis and forensic examination. Implementing appropriate containment/ remediation strategies. </li>
<li> <b> Tier 3  - Threat Hunter: </b> Takes a proactive approach searching for suspicious activity before it becomes an incident and leading network testing such as vulnerability and PEN testing. Also serves as an escalation point for tier 2 analysts analysing complex incidents. </li>
</ul>

### NIST Incident Response Framework

The NIST incident response framework is a four-phase lifecycle, found in NIST’s Special Publication 800-61 [4], intended to help organisations learn how to protect themselves against security incidents. [5] The four stages are as follows: preparation, detection/analysis, containment/eradication, and recovery.

<ol>
<li><b> Preparation: </b> in this phase, incident response plans are generated and CSIRT teams with specified responsibilities are defined to ensure incidents can be effectively prevented and responded to. Proper infrastructure should be set up to enable incident detection and investigation as well as the collection and preservation of evidence. [6] Threat intelligence is crucial to these plans so tier 3 analysts will be vital for this phase. </li>
<li><b>
Detection / Analysis: </b> this phase is the job of the tier 1 analyst and involves collecting data from IT systems, security tools, publicly available information [5] to pinpoint precursors and indicators of a security incident. The signs should then be analysed to determine if they are part of an attack or false positives, and then scored so the incident can be prioritised based on the impact it will have on the company. </li>
<li><b> Containment / Eradication: </b> This phase should be primarily undertaken by a tier 2 analyst with help from a tier 3 analyst in complex cases. Containment is the process of halting the effects of the attack before it causes major or further damage. This usually includes identifying and blocking communication from the attacker’s IP. Once the incident has been contained elements of the incident should be eradicated from affected environments. The systems should be restored quickly to minimise disruption. Logs and events from the incident should be kept for further investigation and to help plan for future attacks. </li>
<li><b> Recovery / Post Incident: </b> This is the phase where reflection should happen and lessons should be learnt. An overview of the incident should be done to determine how well the IR team did, whether the procedures in place were followed and sufficient. And also to analyse what can be done differently and which indicators can be looked into to create a better security posture for the company in the future. [6] </li>
</ol>



## Installation & Data Preparation

## Incident Timeline

| Time (Adjusted to UTC) | Event   | Evidence  |
| --------               | ------- | -------   |
| 09:16:55               | Initial compromise: Bud receives an email notification from AWS Support after the IAM user account web_admin is compromised due to committing the access/ security keys to a public GitHub repository. The adversary gains initial access to AWS. | ![Initial compromise](timeline_aws_email_1.png)|
| 09:16:12 – 09:27:07    | Adversary reconnaissance: The attacker uses the leaked access key to make repeated attempts against IAM resources, generating numerous errors. |
| 13:01:46               | The user bstoll makes the S3 bucket: frothlywebcode publicly available through a misconfiguration. | 
| 13:03:46               | Data exfiltration/staging: A `txt` file (OPEN_BUCKET_PLEASE_FIX.txt) is uploaded by the attacker to confirm the S3 bucket is writable and publicly accessible. |
| 13:04:17               | Data exfiltration/staging: A large `.tar.gz` file is uploaded to the S3 bucket, likely serving as the main payload of the attack. |
| 13:57:54               | The S3 bucket is made private again, closing the vulnerability. |

## Root Cause Analysis

The primary root cause of the incident is the AWS Secret Access key being exposed through human error. At 09:16:55 an automated email from AWS support was received by the user bstoll revealing that this key for the IAM user: web_admin had been leaked. This highly sensitive key had been committed to a publicly available GitHub repository. This allowed a malicious actor to take advantage of this and gain immediate, legitimate and administrative access to the ‘Frothly’ AWS environment.

The secondary root cause was caused by human error and the user bstoll again. After leaking the key, at 13:01:46, bstoll then made the ‘frothlywebcode’ S3 bucket publicly accessible. This allowed the adversary to upload files including malicious ones without any further authorisation, allowing them to stage attack tools for use later on.

Although the incident is a direct consequence of the user: bstoll’s actions, the blame does not fall solely on him as preventative controls should have been in place to prevent these events from happening and having such a detrimental impact on business operations. The IAM user: web_admin which had administrative / high privilege access was not protected by MFA allowing the adversary to log in and make full use of the account without further authentication other than the username and access key. Where MFA was historically considered best practice, it’s now an expectation which needs to be adhered to in order to fully comply with many professional standards and frameworks such as ISO/IEC 27001, CIS Controls v8, and SOC 2 among others. [8] If MFA was mandatory the leaked credentials would have been useless to the attackers. Furthermore, no automated secret scanning tools (such as GitHub pre-commit hooks) were in place within the company. This safeguard would have prevented bstoll from committing the keys to a public repository. In addition to this configuration settings (AWS Account-level Block Public Access) could have been enabled which would have made it impossible for the S3 bucket to be set to public by bstoll preventing the attackers from being able to upload malicious payloads to the bucket.

## Key Indicators of Compromise (IOCs)

| IOC Description | Evidence |
| :--- | :--- |
| **Compromised IAM User Account:** `web_admin`<br>**AWS Access Key:** `AKIAJOGCDXJ5NW5PXUPA`<br>**Secret Key:** `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` | ![IAM_Compromise_Evidence](iam_user.png) |
| **AWS Support Email:** Leak notification with AWS Support Case ID: `5244329601` | ![AWS_Support_emails](AWS_email.png) |
| **Misconfigured S3 Bucket:** `frothlywebcode` | ![S3 bucket](s3_bucket.png) |
| **CloudTrail Event ID:** `ab45689d-69cd-41e7-8705-5350402cf7ac`<br>(API call for S3 configuration change) | ![Event ID](image.png) |
| **Malicious File (Deceptive):** `OPEN_BUCKET_PLEASE_FIX.txt` | ![txt file](txt_file.png) |
| **Malicious File (Main Payload):** `frothly_html_memcached.tar.gz` | ![tar .gz file](tar_file.png) |




## Conclusion
## References 

06/12 <br>
[1] https://www.microsoft.com/en-us/security/business/security-101/what-is-a-security-operations-center-soc <br>
[2] https://radiantsecurity.ai/learn/soc-tier-1-vs-tier-2-vs-tier-3/  <br>
[3] https://www.crowdstrike.com/en-us/cybersecurity-101/next-gen-siem/security-operations-center-soc/ <br>
07/12 <br>
[4]  https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-61r2.pdf <br>
[5] https://www.cynet.com/incident-response/nist-incident-response/ <br>
[6] https://www.crowdstrike.com/en-gb/cybersecurity-101/incident-response/incident-response-steps/ <br>
12/12 <br>
[7] https://www.ibm.com/think/topics/security-operations-center <br>
15/12 <br>
[8] https://pushsecurity.com/blog/how-cyber-breaches-are-driving-tighter-mfa-requirements-and-enforcement/ <br>

## Appendix

## BOTSv3 200-level Questions

Q1.List out the IAM users that accessed an AWS service (successfully or unsuccessfully) in Frothly's AWS environment? <br>
bstoll,btun,splunk_access,web_admin

### Q1 Evidence.

![Q1](<Screenshot 2025-12-01 112623 Q1.png>)
![Q1](<Screenshot 2025-12-01 112931 Q1.png>)
![Q1 Answer](<Screenshot 2025-12-01 113118 Q1 final.png>)


Q2.What field would you use to alert that AWS API activity has occurred without MFA? <br> userIdentity.sessionContext.attributes.mfaAuthenticated

### Q2 Evidence

![Q2](Q2-1.png)
![Q2 Answer](Q2_final-1.png)



Q3.What is the processor number used on the web servers? <br>
E5-2676

### Q3 Evidence

![Q3](Q3.png)
![Q3_final](Q3_final.png)


Q4.Bud accidentally makes an S3 bucket publicly accessible. What is the event ID of the API call that enabled public access? <br>
ab45689d-69cd-41e7-8705-5350402cf7ac



Q5. What is Bud's username? <br>
bstoll


Q6.What is the name of the S3 bucket that was made publicly accessible? <br> 
frothlywebcode


Q7. What is the name of the text file that was successfully uploaded into the S3 bucket while it was publicly accessible? <br> 
OPEN_BUCKET_PLEASE_FIX.txt




Q8. What keywords can you start your search with to help identify what data sources can help you with this? <br>
BSTOLL-L.froth.ly

<!-- ![AWS Support email](image.png) -->

