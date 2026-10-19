# SOC Analyst Home Lab: Real-Time File Integrity Monitoring and Security Alert Investigation Using Wazuh

**Project Category:** Cybersecurity | Security Operations Centre (SOC) | Endpoint Monitoring | Incident Investigation  
**Project Type:** Hands-on Cybersecurity Home Lab  
**Author:** Alex Osale  
**Location:** Dundee, United Kingdom  
**Project Date:** 18–19 October 2026  
**Tools:** Wazuh SIEM/XDR, Ubuntu Linux, Oracle VirtualBox, Linux Command Line

---

## 1. Executive Summary

This project demonstrates the implementation, testing and investigation of File Integrity Monitoring (FIM) using Wazuh within a virtualised cybersecurity home lab.

The objective was to simulate common file-system activities on an Ubuntu Linux endpoint, observe how Wazuh detects those activities, and investigate the resulting security alerts using techniques relevant to a Tier 1 SOC Analyst.

Three controlled activities were successfully tested:

1. File creation
2. File modification
3. File deletion

Wazuh successfully generated alerts for all three activities. The investigation included examining event metadata, monitored file paths, timestamps, file ownership, file sizes and cryptographic hash values.

The project demonstrated the ability to configure and validate endpoint monitoring, investigate security telemetry, interpret file integrity changes and distinguish authorised activity from potentially suspicious behaviour.

**Project Outcome:** All three File Integrity Monitoring tests were successful, and the resulting alerts were investigated using the Wazuh dashboard.

---

## 2. Project Background

File Integrity Monitoring is an important cybersecurity capability used to identify changes to files and directories.

In organisational environments, attackers may attempt to modify sensitive configuration files, create unauthorised scripts, replace legitimate files or delete evidence of their activities.

Security teams use FIM solutions to detect such changes and support investigations.

Wazuh provides File Integrity Monitoring through its Syscheck component, which can monitor file creation, modification and deletion.

This project was designed to develop practical experience in interpreting FIM alerts and understanding their relevance to SOC operations.

---

## 3. Project Objectives

The project aimed to:

- Configure and validate Wazuh FIM on an Ubuntu endpoint.
- Monitor a dedicated Linux directory for file-system changes.
- Generate controlled file creation, modification and deletion events.
- Confirm that Wazuh receives and displays the corresponding alerts.
- Investigate alert metadata and file integrity evidence.
- Compare file hashes before and after modification.
- Understand Linux file permissions and ownership.
- Determine whether detected activities were authorised or suspicious.
- Document findings using a structured SOC investigation approach.

---

## 4. Lab Environment and Architecture

The project was completed in a virtualised home lab using Oracle VirtualBox.

### 4.1 Technologies Used

| Technology | Purpose |
|---|---|
| Oracle VirtualBox | Hosting the virtual machines |
| Ubuntu Linux | Monitored endpoint and test environment |
| Wazuh Server | Collecting and processing security events |
| Wazuh Dashboard | Viewing and investigating alerts |
| Wazuh Agent | Monitoring the Ubuntu endpoint |
| Wazuh Syscheck | Detecting file integrity changes |
| Linux Terminal | Executing test and troubleshooting commands |
| SHA-256, SHA-1 and MD5 | File integrity evidence and hash comparison |

### 4.2 Relevant Network Configuration

| Component | IP Address |
|---|---|
| Wazuh Manager | 192.168.1.11 |
| Ubuntu Endpoint | 192.168.1.12 |
| Ubuntu Wazuh Agent ID | 001 |

The existing virtual lab also included Kali Linux and pfSense for broader networking and cybersecurity practice. The FIM tests documented here were performed locally on Ubuntu.

### 4.3 Monitoring Architecture

Ubuntu Endpoint → Wazuh Agent → Wazuh Manager → Wazuh Dashboard → SOC Analyst Investigation

The Ubuntu endpoint generated file-system events. The Wazuh agent monitored the configured directory and forwarded relevant information to the Wazuh manager.

The resulting alerts were examined through the Wazuh dashboard.

---

## 5. File Integrity Monitoring Configuration

A dedicated directory was selected for the FIM tests:

`/home/ubuntu/fim-lab`

The Wazuh agent configuration file was accessed using:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

The Syscheck configuration was used to enable File Integrity Monitoring.

The lab directory was configured for real-time monitoring using the following configuration entry:

```xml
<directories realtime="yes" check_all="yes">/home/ubuntu/fim-lab</directories>
```

This configuration directs Wazuh to monitor the selected directory for changes and collect available file integrity attributes.

The Wazuh dashboard subsequently confirmed that the test events were detected in real-time mode.

**Evidence:** Screenshot of the Wazuh agent configuration showing the monitored directory.

---

## 6. Practical Testing and Implementation

### 6.1 Test One: File Creation Detection

**Objective:** Verify that Wazuh detects the creation of a new file within the monitored directory.

A test file was created using the following command:

```bash
echo "SOC FIM Test" | sudo tee /home/ubuntu/fim-lab/test.txt
```

The command created a file named `test.txt` containing the text `SOC FIM Test`.

The Wazuh dashboard was then examined under File Integrity Monitoring → Events.

**Observed Result:**

Wazuh generated a file creation alert.

The alert identified:

- Agent: Ubuntu
- Agent ID: 001
- File path: `/home/ubuntu/fim-lab/test.txt`
- Decoder: `syscheck_new_entry`
- Rule description: File added to the system
- Monitoring mode: Realtime

**Outcome: PASS**

The test confirmed that Wazuh detected the creation of a file in the monitored directory.

**Evidence:** Screenshot 1 — Wazuh File Creation Alert.

### 6.2 Test Two: File Modification Detection

**Objective:** Verify that Wazuh detects modifications to an existing monitored file.

The following command was used:

```bash
echo "Modified by SOC Analyst" >> /home/ubuntu/fim-lab/test.txt
```

The `>>` operator appends text to an existing file without replacing its previous contents.

Initially, the command returned a permission-denied error.

Investigation revealed that the file was owned by the root account because it had originally been created using `sudo tee`.

The file ownership was corrected using:

```bash
sudo chown ubuntu:ubuntu /home/ubuntu/fim-lab/test.txt
```

After correcting the ownership, the modification test was completed successfully.

**Observed Result:**

Wazuh generated file modification alerts for the monitored file.

The dashboard showed the event type:

`syscheck.event: modified`

The investigation confirmed that the file's size and cryptographic hashes had changed.

**Outcome: PASS**

The test demonstrated that Wazuh could identify changes to an existing monitored file.

**Evidence:** Screenshot 2 — Wazuh File Modification Alerts.

### 6.3 Test Three: File Deletion Detection

**Objective:** Verify that Wazuh detects the deletion of a monitored file.

The following command was used:

```bash
rm /home/ubuntu/fim-lab/test.txt
```

An initial permission-denied error highlighted the distinction between file ownership and directory permissions.

The directory permissions and ownership were investigated as part of the troubleshooting process.

The deletion was subsequently completed, and the Wazuh dashboard displayed a corresponding deletion event.

**Observed Result:**

- File path: `/home/ubuntu/fim-lab/test.txt`
- Event: `deleted`
- Decoder: `syscheck_deleted`
- Rule description: File deleted
- Monitoring mode: Realtime
- Alert timestamp: 18 October 2026, 23:46:22

**Outcome: PASS**

The test confirmed that Wazuh detected the removal of a monitored file.

**Evidence:** Screenshot 3 — Wazuh File Deletion Alert.

---

## 7. SOC Alert Investigation

Following successful detection testing, the generated alerts were investigated using Wazuh's Document Details interface.

The investigation focused on determining:

- Which endpoint generated the alert.
- Which file was affected.
- What file-system activity occurred.
- When the activity was detected.
- Whether file integrity attributes had changed.
- Whether the activity was authorised.

### 7.1 File Deletion Investigation

The deletion alert provided the following evidence:

| Investigation Field | Finding |
|---|---|
| Agent | Ubuntu |
| Agent ID | 001 |
| Agent IP | 192.168.1.12 |
| Wazuh Manager | wazuh-server |
| File Path | `/home/ubuntu/fim-lab/test.txt` |
| Event Type | deleted |
| Monitoring Mode | realtime |
| Rule Description | File deleted |
| Alert Timestamp | 18 October 2026, 23:46:22 |
| Recorded File Owner | ubuntu |
| Recorded File Size | 37 bytes |

The evidence established that Wazuh had detected a deletion event affecting the monitored file.

The alert did not independently establish which user initiated the deletion or whether the action was malicious.

Because the deletion was deliberately performed as part of the authorised lab test, the event was classified as benign test activity.

**Disposition:** Authorised activity — no security incident confirmed.

### 7.2 File Modification Investigation

The modification alert contained both pre-modification and post-modification file integrity attributes.

| Attribute | Before Modification | After Modification |
|---|---|---|
| File Size | 13 bytes | 37 bytes |
| Modification Time | 23:07:57 | 23:32:23 |
| SHA-256 | Original hash | Changed hash |
| SHA-1 | Original hash | Changed hash |
| MD5 | Original hash | Changed hash |

The file size increased by 24 bytes.

The SHA-256, SHA-1 and MD5 values also differed between the before and after states.

These findings provided strong evidence that the contents of the monitored file had changed.

The Wazuh modification alert was recorded on 18 October 2026 at 23:32:31.

**Disposition:** Authorised file modification — benign lab activity.

---

## 8. Understanding File Hashes in Security Investigations

A cryptographic hash is a fixed-length value calculated from data.

SHA-256 is commonly used in security investigations to identify files and compare their contents.

In this project, Wazuh recorded SHA-256 values before and after modification.

The differing values demonstrated that the file's contents had changed.

This is important because attackers may alter files without changing their names or locations.

Hash comparisons help analysts identify changes that might otherwise be overlooked.

However, a changed hash alone does not establish malicious intent.

Further investigation may require examining user activity, process execution, authentication logs, authorised change records and threat intelligence.

This project demonstrated hash comparison and integrity analysis. It did not establish the presence of malware.

---

## 9. Challenges Encountered and Solutions

### Challenge 1: Permission Denied During File Modification

**Problem:** Ubuntu prevented modification of the test file.

**Cause:** The file was owned by `root`, while the command was executed as the standard `ubuntu` user.

**Investigation Command:**

```bash
ls -l /home/ubuntu/fim-lab/test.txt
```

**Solution:**

```bash
sudo chown ubuntu:ubuntu /home/ubuntu/fim-lab/test.txt
```

**Lesson Learned:** Linux file ownership and permissions directly influence which users can modify files.

### Challenge 2: Permission Denied During File Deletion

**Problem:** The file deletion command initially failed.

**Likely Cause:** The user account lacked sufficient permission on the parent directory.

**Investigation Command:**

```bash
ls -ld /home/ubuntu/fim-lab
```

**Lesson Learned:** Deleting a file depends on permissions on its containing directory, not only the permissions of the file itself.

### Challenge 3: Interpreting Wazuh Alerts

**Problem:** Multiple alerts were displayed for the same monitored file.

**Approach:** The Wazuh Events interface and Document Details view were used to distinguish file creation, modification and deletion events.

**Lesson Learned:** Analysts must inspect event-specific metadata rather than assuming that all alerts relating to the same file represent the same activity.

### Challenge 4: Understanding Hash Evidence

**Problem:** The modification alert displayed several cryptographic hash values.

**Approach:** The before and after file sizes, modification timestamps and hash values were compared.

**Lesson Learned:** Hash comparisons can establish file integrity changes, but additional context is needed to determine intent.

---

## 10. Results and Findings

| Test | Expected Result | Observed Result | Status |
|---|---|---|---|
| File Creation | Wazuh detects a new file | File added alert generated | PASS |
| File Modification | Wazuh detects changes to a file | Modification alerts generated | PASS |
| File Deletion | Wazuh detects file removal | File deleted alert generated | PASS |
| File Integrity Analysis | Wazuh provides file metadata | File sizes, timestamps and hashes examined | PASS |
| Alert Investigation | Analyst interprets event evidence | Creation, modification and deletion events reviewed | PASS |
| Event Classification | Determine whether activity was authorised | Lab events classified as benign | PASS |

### Key Findings

1. Wazuh successfully monitored the selected Ubuntu directory.
2. File creation, modification and deletion were detected.
3. File modification alerts provided evidence of changes in file size and cryptographic hashes.
4. Linux file permissions affected the execution of test commands.
5. The Wazuh dashboard provided useful telemetry for investigating file-system activity.
6. The observed events were authorised lab activities, not confirmed cyberattacks.

---

## 11. SOC Analyst Skills Demonstrated

This project provided practical experience in:

**Security Monitoring:** Monitoring an Ubuntu endpoint through Wazuh.

**Alert Triage:** Identifying the affected endpoint, file path and event type.

**Log Analysis:** Interpreting Wazuh event fields, timestamps and rule descriptions.

**File Integrity Monitoring:** Detecting file creation, modification and deletion.

**Digital Forensics Fundamentals:** Comparing file sizes, timestamps and cryptographic hashes.

**Linux Administration:** Troubleshooting file ownership and directory permissions.

**Incident Investigation:** Distinguishing confirmed file-system activity from unverified malicious intent.

**Security Documentation:** Recording evidence, investigation findings and event dispositions.

---

## 12. Limitations and Future Improvements

The activities were performed in a controlled lab environment.

Although Wazuh successfully detected the changes, the project did not demonstrate a confirmed malicious compromise.

The FIM alerts did not independently identify the process or user responsible for every file operation.

Future improvements could include:

- Simulating unauthorised file changes in an isolated lab.
- Correlating FIM events with Linux audit logs.
- Investigating suspicious processes and authentication activity.
- Performing threat-intelligence enrichment using file hashes.
- Developing custom Wazuh detection rules.
- Practising alert escalation and incident response.
- Testing remediation and post-remediation verification.

These activities would extend the project into a more complete SOC detection and response exercise.

---

## 13. Conclusion

This project successfully demonstrated the practical use of Wazuh File Integrity Monitoring to detect and investigate file-system changes on an Ubuntu Linux endpoint.

Three controlled tests were performed: file creation, file modification and file deletion. All three generated corresponding Wazuh alerts.

The investigation involved examining event metadata, timestamps, file permissions, ownership information, file sizes and cryptographic hash values.

The project reinforced the importance of distinguishing a security alert from a confirmed security incident.

It also strengthened practical skills in endpoint monitoring, Linux administration, security event investigation and evidence-based documentation.

**Overall Result: Successful implementation, validation and investigation of Wazuh File Integrity Monitoring in a virtualised SOC home lab.**

---

## 14. Project Evidence

The following screenshots should accompany the GitHub project:

| Evidence | Screenshot Description |
|---|---|
| Figure 1 | Wazuh File Creation Alert — `syscheck_new_entry` |
| Figure 2 | Wazuh File Modification Events — `modified` |
| Figure 3 | Wazuh File Deletion Event — `deleted` |
| Figure 4 | File Modification Investigation — SHA-256 Before and After |
| Figure 5 | File Deletion Investigation — Document Details |

---

## 15. Author

**Alex Osale**  
Aspiring Cybersecurity Analyst | SOC Operations | Network Security | Linux Security

**Technical Skills:** Wazuh, Ubuntu Linux, Kali Linux, Wireshark, Cisco Packet Tracer, pfSense, VirtualBox, File Integrity Monitoring, Security Event Analysis and Network Troubleshooting.

**Career Objective:** To develop practical cybersecurity expertise and progress into a professional Cybersecurity Analyst role.
