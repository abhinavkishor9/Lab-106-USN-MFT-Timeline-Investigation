# Lab-106-USN-MFT-Timeline-Investigation
## Overview
Master File Table (MFT):
The MFT maintains file and directory records on an NTFS volume. Records can contain file names, timestamps, file attributes, and references to other records. Forensic tools can parse these records to examine file-system metadata.

USN Change Journal
The USN journal records information about changes to files and directories on an NTFS volume. Records can include a file reference number, parent reference, timestamp, and reason flags such as file creation, data modification, rename, or deletion.

Why correlate them?
- The USN journal can help identify recorded changes and their sequence.
- MFT records can provide additional file-system metadata and file-reference information.
- Correlating file references, names, paths, and timestamps can help reconstruct a more reliable timeline.
- Neither artifact alone necessarily establishes who performed an action, whether a file was executed, or whether activity was malicious.

This lab establishes a controlled file-activity scenario and records the NTFS journal state, test operations, available endpoint telemetry, and evidence files. The objective is to build a defensible timeline without assuming that an artifact alone proves user intent or malicious activity.
## Lab Environment
| Component | Configuration |
|---|---|
| Endpoint | `DESKTOP-9MMM37V` |
| Operating system | Windows 11 |
| Shell | PowerShell 7 |
| Target volume | `C:` |
| File system | NTFS |
| Volume health | Healthy |
| Evidence directory | `C:\USNMFTTimelineLab\Evidence` |
| Test directory | `C:\USNMFTTimelineLab\TestFiles` |
| Endpoint telemetry | Sysmon |
| SIEM | Wazuh |

## Lab Scenario
A SOC analyst is investigating file-system activity on a Windows endpoint and needs to reconstruct a sequence of file operations using available NTFS forensic artifacts. The investigation focuses on the Update Sequence Number (USN) Change Journal and Master File Table (MFT), supported by Sysmon and Wazuh telemetry where relevant.
To establish a known sequence of activity, the analyst creates a dedicated test directory at `C:\USNMFTTimelineLab\TestFiles` and performs several controlled operations. The sequence includes creating `document-A.txt`, appending additional content, copying it to `document-B.txt`, renaming the copy to `document-renamed.txt`, and deleting the renamed file. These known actions provide reference points for evaluating the file-system evidence.
The analyst first verifies that the `C:` volume uses NTFS and records the current USN journal configuration. The journal is then queried, and its output is redirected to an evidence file. The analyst also exports metadata for files remaining in the test directory and generates a SHA-256 manifest to support evidence integrity checks.
The next stage is to examine the collected journal records and acquire and parse MFT metadata using a suitable forensic method. The analyst must determine whether the available records contain the expected file names, file-reference numbers, parent references, timestamps, and USN reason flags. Relevant Sysmon events and Wazuh records are also examined to identify supporting evidence of file operations and, where possible, associated processes.
During analysis, the analyst must account for incomplete collection output, unrelated system activity, timestamp differences, and the possibility that some records are unavailable or cannot be confidently correlated. The presence of a Sysmon event or a file-system timestamp alone is insufficient to establish that an event belongs to the controlled test sequence.
### Investigation Questions
- Does the collected USN journal output contain records associated with the controlled file operations?
- Which USN reason flags and timestamps support the creation, modification, rename, and deletion sequence?
- Can the relevant file-reference and parent-reference values be correlated with parsed MFT records?
- Does the final directory listing agree with the operations performed, and what evidence may remain for the deleted file?
- Do any Sysmon or Wazuh events provide meaningful corroboration of the test activity?
- Are there collection gaps, timestamp inconsistencies, or other limitations that affect the reliability of the reconstructed timeline?
- 
### Expected Outcome
The analyst should produce a documented file-activity timeline supported by validated USN records, MFT metadata where successfully acquired, and relevant endpoint telemetry. The investigation must distinguish confirmed file-system observations from correlations that remain unverified and clearly document any missing artifacts or collection limitations.## Investigation Objectives
- Verify that the target volume is NTFS and record its relevant properties.
- Record the USN Change Journal configuration and its identifier.
- Perform controlled file-system operations in a dedicated directory.
- Collect the USN journal output and preserve it as evidence.
- Record metadata for files remaining after the test operations.
- Inspect Sysmon and Wazuh telemetry for related activity.
- Acquire and parse MFT metadata using a suitable forensic method.
- Correlate file-reference information, timestamps, names, and USN reason flags where available.
- Generate SHA-256 hashes for evidence files.
- Document collection limitations and distinguish observations from conclusions.

## Lab Objectives
The objective of this lab is to investigate NTFS file-system activity by examining the USN Change Journal, analyzing MFT metadata, and correlating relevant artifacts with supporting endpoint telemetry to reconstruct a reliable file-activity timeline.
- **Examine NTFS forensic artifacts:** Understand the purpose of the USN Change Journal and Master File Table (MFT), including the types of file-system information each can provide during an investigation.
- **Validate the target volume:** Confirm the file system type, volume health, and availability of the USN journal before collecting evidence.
- **Establish a controlled activity sequence:** Perform known file creation, modification, copying, renaming, and deletion operations within a dedicated test directory.
- **Collect USN journal evidence:** Record journal configuration details and capture available records for subsequent examination.
- **Analyze file-system metadata:** Acquire and parse MFT metadata using a suitable forensic method to identify file records, file-reference numbers, parent references, and relevant timestamps.
- **Reconstruct file activity:** Examine USN reason flags and correlate them with the controlled file operations to establish the sequence of recorded changes.
- **Correlate USN and MFT artifacts:** Compare file-reference information, names, parent relationships, and timestamps to determine whether the artifacts support a consistent account of file-system activity.
- **Investigate supporting endpoint telemetry:** Examine available Sysmon and Wazuh events for relevant file operations and process details without assuming that unrelated events belong to the investigation.
- **Validate timeline accuracy:** Compare artifact timestamps with the known test sequence, account for timezone differences, and identify gaps or inconsistencies that could affect interpretation.
- **Preserve evidence integrity:** Maintain original collection outputs, document forensic tools and collection limitations, and generate SHA-256 hashes for evidence files.
- **Evaluate forensic limitations:** Assess how journal retention, missing records, incomplete metadata, and file-reference reuse may affect timeline reconstruction.
- **Document defensible conclusions:** Separate directly observed artifacts, corroborated findings, and unresolved hypotheses in the final investigation.
## Expected Learning Outcome
Upon completing this lab, the investigator should be able to examine NTFS journal records and MFT metadata, correlate file-system artifacts with known activity, develop a defensible timeline, and explain the limitations of conclusions that rely on incomplete or uncorroborated evidence.


## Controlled File Operations
The test sequence used the following operations:
1. Create `document-A.txt` with initial content.
2. Append additional content to `document-A.txt`.
3. Copy the file to `document-B.txt`.
4. Rename the copy to `document-renamed.txt`.
5. Delete `document-renamed.txt`.
The provided commands do not delete `document-A.txt`. Based on the sequence, that file should remain in the test directory; the contents of `remaining-files.csv` should be checked to confirm the observed final state.
## Evidence Artifacts
| Artifact | Purpose |
|---|---|
| `usn-journal-baseline.txt` | Records the USN journal configuration reported by `fsutil` |
| `usn-journal-output.txt` | Captures the USN journal read output |
| `remaining-files.csv` | Records metadata for files remaining in the test directory |
| `evidence-hashes.csv` | Stores SHA-256 hashes generated for the other evidence files |
Additional artifacts, including parsed MFT output and a consolidated timeline, should be added when the corresponding acquisition and analysis steps are completed.
## Preliminary Findings
- The `C:` volume was reported as NTFS with a healthy status.
- The USN journal query returned a journal identifier, first USN, next USN, maximum size, and allocation delta.
- The journal reported a maximum size of 32 MB and an allocation delta of 8 MB.
- The controlled operations created and modified `document-A.txt`, copied it, renamed the copy, and deleted the renamed copy.
- The USN read command was executed with output redirected to `usn-journal-output.txt`. The supplied screenshots do not show the contents of that file, so the presence and completeness of the expected records still require verification.
- A Sysmon event document was visible in Wazuh. The supplied excerpt included a system timestamp and `task: 11`, but did not include enough fields to establish its event ID or correlation to the test operations.
- An MFT acquisition or parsed MFT export was not shown in the supplied evidence.
## Investigation Limitations
The available evidence does not yet establish a complete USN-to-MFT correlation. The journal output must be inspected for records matching the controlled file operations, and MFT metadata must be acquired and parsed before file-reference relationships can be assessed.
Sysmon and Wazuh can provide supporting telemetry, but neither should be treated as a substitute for the raw USN journal or MFT metadata. An event without a matching file path, timestamp, or other relevant identifier cannot automatically be attributed to the lab.
File-system timestamps also require careful interpretation. Different timestamp sources may reflect different events, and some metadata can be altered by normal file operations or other processes.
