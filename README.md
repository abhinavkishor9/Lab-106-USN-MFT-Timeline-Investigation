# Lab 106 — USN + MFT Timeline Investigation
## Repository Description
Investigate NTFS file-system activity using the USN Change Journal, MFT metadata, and supporting Sysmon/Wazuh telemetry. Reconstruct controlled file operations, preserve forensic artifacts, correlate timestamps, and document evidence limitations.
## Overview
The NTFS file system maintains metadata and change records that can help investigators reconstruct file activity. Two important forensic artifacts are the Master File Table (MFT) and the Update Sequence Number (USN) Change Journal.
The MFT contains file and directory records with metadata such as file names, timestamps, attributes, and file-reference information. The USN Change Journal records information about changes made to files and directories, including reasons for changes such as creation, data modification, renaming, and deletion.
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
## Investigation Scenario
A SOC analyst needs to reconstruct a sequence of file-system operations on a Windows endpoint. The analyst creates a dedicated test directory and performs controlled file creation, content modification, copying, renaming, and deletion.
The investigation records the USN journal configuration, captures journal output, preserves a listing of the remaining test files, and examines a Sysmon event indexed in Wazuh. These artifacts are intended to support a timeline of the controlled activity.
The analyst must distinguish the known test operations from unrelated endpoint events and determine whether the collected artifacts provide sufficient information to correlate the file operations.
## Investigation Objectives
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
## Conclusion
The lab established an NTFS test environment, recorded the USN journal configuration, performed controlled file operations, collected journal output, captured remaining-file metadata, and generated a SHA-256 manifest.
The next step is to validate the journal output, acquire and parse MFT metadata, and correlate the resulting records with the known test sequence. Until this is completed, the investigation should be described as a preliminary USN and supporting-telemetry collection rather than a completed MFT timeline reconstruction.
## Key Learning
**A file-system artifact is evidence of recorded state or activity, not a complete explanation of what happened.** Reliable timeline reconstruction requires validated artifacts, defensible correlation, and clear documentation of uncertainty.
