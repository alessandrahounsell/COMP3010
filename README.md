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

![NIST Incident Response Cycle](<NIST_phases.png>)
NIST Incident Response Cycle [9]

### Reflection on how incident handling and methodologies are used within the BOTsV3 Data

| Question | Phase | Team Member | SOC Relevance |
|---------|-------|-------------|---------------|
| Q1 | Detection / Analysis | Tier 1 | Tier 1 analysts establish a baseline for normal activity, such as which accounts are active and what is normal for the user. This allows for better anomaly detection. Log management creates a collection of log data generated by each network event. The list of IAM users is also used by the SOC to enforce the principle of least privilege. High-privilege users such as `splunk_access` and `web_admin` require more scrutiny, as compromise of these accounts could cause significant harm to the business. |
| Q2 | Preparation | Tier 3 | An action completed without MFA authentication is a potential policy violation. A Tier 3 analyst helps define security standards, such as making MFA mandatory during the preparation phase. This mechanism allows the SOC to detect and alert on policy failures, preventing account compromise. |
| Q3 | Preparation | Tier 3 | Knowing the processor number is essential for vulnerability management, as it allows the SOC to respond quickly if a hardware-level vulnerability is announced. Logging this information during the preparation phase helps determine exposure, prioritise patching, and monitor for exploitation attempts. |
| Q4 | Detection / Analysis | Tier 2 | This event ID represents the starting point of the incident and serves as a crucial reference for a Tier 2 incident responder when creating an accurate incident timeline. |
| Q5 | Detection / Analysis | Tier 2 | Identifying *Bstoll* as the user who made the error is key for root cause analysis. The Tier 2 analyst uses this as a pivot point for investigation, questioning why the action was performed, whether it should have been allowed, and if IAM permissions align with least privilege principles. |
| Q6 | Detection / Analysis | Tier 2 | Correctly identifying the affected asset allows the Tier 2 analyst to assess the incident’s scope and prioritise containment. The bucket name suggests it contains source code for the Frothly website, elevating the incident’s severity due to risks of intellectual property theft or data leakage. |
| Q7 | Detection / Analysis | Tier 2 | Confirmation that the file was successfully uploaded verifies the bucket was publicly accessible. This information feeds into the impact assessment conducted by the Tier 2 analyst following containment. |
| Q8 | Detection / Analysis | Tier 1 & Tier 2 | Keywords must be correlated across multiple log sources by both Tier 1 and Tier 2 analysts to enrich alert data and improve detection accuracy. |
| All Qs | Response (Containment / Eradication) | Tier 2 | Using findings from the detection and analysis phase, along with enriched data from Tier 1 analysts, the Tier 2 analyst must immediately implement a containment plan. This includes making the S3 bucket private and suspending leaked credentials to prevent further lateral movement. |
| Q2, Q5 | Recovery | All Tiers | Tier 3 analysts lead recovery efforts by hardening the environment. This includes enforcing stricter IAM permissions and mandatory MFA to strengthen the organisation’s overall security posture. |




## Installation & Data Preparation

This guide outlines the process for installing Splunk Enterprise on an Ubuntu VM, applying a license, and ingesting the BOTSv3 dataset.

### Splunk Installation

I installed Splunk onto an Ubuntu VM using the following steps:

1.  **Download:** Log into Splunk Enterprise and navigate to the **Splunk Enterprise downloads page**. Select **Linux** and locate the `.tgz` version.
![Splunk downloads page](<Installation/splunk_download.png>)
2.  Click **'Copy wget link'** to copy the command to your clipboard.
![Copy link](<Installation/splunk_download_2.png>)
3.  **Transfer:** Open the terminal in the VM, run `cd Desktop`, and paste the `wget` command.
![Paste wget command](<Installation/splunk_install_3.png>)
4.  **Extract:** Once the download is finished, install Splunk into the `/opt` directory using:
    ```bash
    sudo tar xvzf splunk-10.0.1-c486717c322.b-linux-amd64.tgz -C /opt
    ```
    ![Extract](<Installation/splunk_install_4.png>)
5.  **Navigate:** Once the extraction is complete, move to the bin directory:
    ```bash
    cd /opt/splunk/bin
    ```
6.  **Initialise:** Start Splunk and accept the license:
    ```bash
    sudo ./splunk start --accept-license
    ```
    ![Init](<Installation/splunk_install_start_5.png>)
7.  **Setup Account:** Enter an admin username and password when prompted to create your account.
![Setup admin](<Installation/splunk_set_up_admin_6.png>)
8.  **Access Web UI:** Once Splunk is running, navigate to `http://localhost:8000` in your browser.
![Localhost](<Installation/splunk_localhost_8.png>)
9. **Management Commands:**
    * **Start:** `sudo ./splunk start`
    * **Stop:** `sudo ./splunk stop`

### Adding Splunk License

1.  Download the license file from the DLE within the VM.
![Download license](<Installation/license_1.png>)
2.  With Splunk running, navigate to **Settings > Licensing**.
![Download license](<Installation/license_2.png>)
3.  Click **Add License** and upload the file from your `Downloads` folder.
![Add license](<Installation/Add_license.png>)
4. Click **Install**
![Download license](<Installation/license_3.png>)
4.  Restart Splunk to apply changes
![License restart](<Installation/license_restart_4.png>)

### Installing the BOTSv3 Dataset

The BOTSv3 dataset is publicly available on [GitHub](https://github.com/splunk/botsv3).

1.  Download and extract the dataset within the Linux VM.
![Extract dataset](<Installation/extract_dataset.png>)

2.  Open the terminal and elevate permissions: `sudo su`.
3.  Navigate to the dataset location (e.g., `cd /home/student/Downloads`).
![Dataset location](<Installation/dataset.png>)

4.  **Copy to Splunk Apps:**
    ```bash
    cp -r botsv3_data_set /opt/splunk/etc/apps
    ```
5.  Navigate back to the Splunk bin directory:
    ```bash
    cd /opt/splunk/bin
    ```
6.  Start Splunk and log in to the web interface.
![Dataset set up](<Installation/dataset_2.png>)
7.  Navigate to **Search & Reporting**.
8.  **Query Data:** Enter the following search query:
    ```splunk
    index=botsv3 earliest=0
    ```
9.  Set the time range to **All time**.
10. **Validation:** Click search. There should be **2,083,056 events** within the dataset.
![Validate Dataset set up](<Installation/dataset_validate.png>)

### Installing Windows Add-on
Used to investigate coin mining events occurring on Windows machines.

1.  Download the **Splunk Add-on for Microsoft Windows** from [Splunkbase](https://splunkbase.splunk.com/app/742).
![Windows Add on](<Installation/windows_add_on.png>)
2.  Log in to the Splunk web interface.
3.  Go to **Apps > Manage Apps** and click **Install app from file**.
4.  Upload the downloaded file.
![Windows Add on](<Installation/Install_windows_add_on_2.png>)
5.  Restart Splunk to initialise the new fields and data models.


## Incident Timeline

| Time (Adjusted to UTC) | Event   | Evidence  |
| --------               | ------- | -------   |
| 09:16:55               | Initial compromise: Bud receives an email notification from AWS Support after the IAM user account web_admin is compromised due to committing the access/ security keys to a public GitHub repository. The adversary gains initial access to AWS. | ![Initial compromise](<Timeline/timeline_aws_email_1.png>)|
| 09:16:12 – 09:27:07    | Adversary reconnaissance: The attacker uses the leaked access key to make repeated attempts against IAM resources, generating numerous errors. |  ![Adversary reconnaissance](<Timeline/timeline_accesskey_attempts_2.png>)|
| 13:01:46               | The user bstoll makes the S3 bucket: frothlywebcode publicly available through a misconfiguration. | ![S3 bucket set to public](<Timeline/.png>)|
| 13:03:46               | Data exfiltration/staging: A `txt` file (OPEN_BUCKET_PLEASE_FIX.txt) is uploaded by the attacker to confirm the S3 bucket is writable and publicly accessible. | ![.txt upload](<Timeline/txt_file_4.png>)|
| 13:04:17               | Data Exfiltration/Staging: A large .tar.gz file is uploaded to the S3 bucket which is likely the main payload for the attack containing a miner. | ![Main payload upload](<Timeline/staging_file_5.png>)|
| 13:37:51                | The endpoint BSTOLL-L begins executing the payload and shows signs of coin mining activity with the CPU reaching 100% utilisation. | ![Main payload upload](<Timeline/cpu_utalisation_6.png>)|
| 13:57:54               | The S3 bucket is made private again, closing the vulnerability. | ![S3 bucket made private](<Timeline/timeline_7.png>)|

## Root Cause Analysis

The primary root cause of the incident is the AWS Secret Access key being exposed through human error. At 09:16:55 an automated email from AWS support was received by the user bstoll revealing that this key for the IAM user: web_admin had been leaked. This highly sensitive key had been committed to a publicly available GitHub repository. This allowed a malicious actor to take advantage of this and gain immediate, legitimate and administrative access to the ‘Frothly’ AWS environment.

The secondary root cause was caused by human error and the user bstoll again. After leaking the key, at 13:01:46, bstoll then made the ‘frothlywebcode’ S3 bucket publicly accessible. This allowed the adversary to upload files including malicious ones without any further authorisation, allowing them to stage attack tools for use later on.

Although the incident is a direct consequence of the user: bstoll’s actions, the blame does not fall solely on him as preventative controls should have been in place to prevent these events from happening and having such a detrimental impact on business operations. The IAM user: web_admin which had administrative / high privilege access was not protected by MFA allowing the adversary to log in and make full use of the account without further authentication other than the username and access key. Where MFA was historically considered best practice, it’s now an expectation which needs to be adhered to in order to fully comply with many professional standards and frameworks such as ISO/IEC 27001, CIS Controls v8, and SOC 2 among others. [8] If MFA was mandatory the leaked credentials would have been useless to the attackers. Furthermore, no automated secret scanning tools (such as GitHub pre-commit hooks) were in place within the company. This safeguard would have prevented bstoll from committing the keys to a public repository. In addition to this configuration settings (AWS Account-level Block Public Access) could have been enabled which would have made it impossible for the S3 bucket to be set to public by bstoll preventing the attackers from being able to upload malicious payloads to the bucket.

## Key Indicators of Compromise (IOCs)

| IOC Description | Evidence |
| :--- | :--- |
| **Compromised IAM User Account:** `web_admin`<br>**AWS Access Key:** `AKIAJOGCDXJ5NW5PXUPA`<br>**Secret Key:** `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` | ![IAM_Compromise_Evidence](<IOCs/iam_user.png>) |
| **AWS Support Email:** Leak notification with AWS Support Case ID: `5244329601` | ![AWS_Support_emails](<IOCs/AWS_email.png>) |
| **Misconfigured S3 Bucket:** `frothlywebcode` | ![S3 bucket](<IOCs/s3_bucket.png>) |
| **CloudTrail Event ID:** `ab45689d-69cd-41e7-8705-5350402cf7ac`<br>(API call for S3 configuration change) | ![Event ID](<IOCs/Event_id.png>) |
| **Malicious File (Deceptive):** `OPEN_BUCKET_PLEASE_FIX.txt` | ![txt file](<IOCs/txt_file.png>) |
| **Malicious File (Main Payload):** `frothly_html_memcached.tar.gz` | ![tar .gz file](<IOCs/tar_gz_file.png>) |

### Recovery Timeline

| Time (UTC) | Activity | Evidence |
| :--- | :--- | :--- |
| 13:57:54 | The exposed S3 bucket is set to private. | ![S3 bucket made private](<Timeline/bucket_set_false.png>) |
| 14:05:23 | Mining process is terminated on BSTOLL-L endpoint. | ![mining terminated](<Timeline/mining_ends.png>) |

After the incident is contained the malicious files should be eradicated from the system and the company should start recovery to return back to business as usual.

### Suggested timeline for eradication and recovery:

<ul>
<li>14:30 - The malicious files: PEN_BUCKET_PLEASE_FIX.txt and frothly_html_memcached.tar.gz, the main payload should be deleted from the frothlywebcode S3 bucket.
<li>15:00 - The host: BSTOLL-L should be promptly isolated to prevent further cryptomining and lateral movement.
<li>16:00 - CloudTrail logs should be reviewed for the 24-hour window to ensure no other backdoors (like new IAM users) were created.
<li>The following day - BAU, Automated deployments and pipeline should be re-enabled.
</ul>


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
22/12 <br>
[9] https://auditboard.com/blog/nist-incident-response 

## Appendix

## BOTSv3 200-level Questions

Q1.List out the IAM users that accessed an AWS service (successfully or unsuccessfully) in Frothly's AWS environment? <br>
bstoll,btun,splunk_access,web_admin

### Q1 Evidence

![Q1](<Evidence for BOTSv3 200-level Questions/Q1.png>)
![Q1](<Evidence for BOTSv3 200-level Questions/Q1-1.png>)
![Q1 Answer](<Evidence for BOTSv3 200-level Questions/Q1_final.png>)


Q2.What field would you use to alert that AWS API activity has occurred without MFA? <br> userIdentity.sessionContext.attributes.mfaAuthenticated

### Q2 Evidence

![Q2](<Evidence for BOTSv3 200-level Questions/Q2-1.png>)
![Q2 Answer](<Evidence for BOTSv3 200-level Questions/Q2_final-1.png>)



Q3.What is the processor number used on the web servers? <br>
E5-2676

### Q3 Evidence

![Q3](<Evidence for BOTSv3 200-level Questions/Q3.png>)
![Q3_final](<Evidence for BOTSv3 200-level Questions/Q3_final.png>)


Q4.Bud accidentally makes an S3 bucket publicly accessible. What is the event ID of the API call that enabled public access? <br>
ab45689d-69cd-41e7-8705-5350402cf7ac

### Q4 Evidence

![Q4](<Evidence for BOTSv3 200-level Questions/Q4.png>)
![Q4_final](<Evidence for BOTSv3 200-level Questions/Q4_final.png>)


Q5. What is Bud's username? <br>
bstoll

### Q5 Evidence 

![Q5](<Evidence for BOTSv3 200-level Questions/Q5.png>)

Q6.What is the name of the S3 bucket that was made publicly accessible? <br> 
frothlywebcode

### Q6 Evidence

![Q6](<Evidence for BOTSv3 200-level Questions/Q6.png>)



Q7. What is the name of the text file that was successfully uploaded into the S3 bucket while it was publicly accessible? <br> 
OPEN_BUCKET_PLEASE_FIX.txt

### Q7 Evidence

![Q7](<Evidence for BOTSv3 200-level Questions/Q7.png>)
![Q7-1](<Evidence for BOTSv3 200-level Questions/Q7-1.png>)
![Q7 Final](<Evidence for BOTSv3 200-level Questions/Q7_final.png>)


Q8. What keywords can you start your search with to help identify what data sources can help you with this? <br>
BSTOLL-L.froth.ly

### Q8 Evidence

![Q8](<Evidence for BOTSv3 200-level Questions/Q8.png>)
![Q8-1](<Evidence for BOTSv3 200-level Questions/Q8-1.png>)
![Q7 Final](<Evidence for BOTSv3 200-level Questions/Q8_final.png>)

