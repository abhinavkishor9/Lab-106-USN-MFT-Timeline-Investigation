# Lab 106 — USN + MFT Timeline Investigation
## Timeline Overview
This timeline records the known setup, controlled file operations, and artifact-collection steps for the investigation on `DESKTOP-9MMM37V`.
Exact timestamps for most PowerShell operations were not included in the supplied screenshots. They are therefore listed without invented times. The Wazuh timestamp is recorded separately and has not been attributed to the test operations.
## Timeline
| Time | Event | Evidence | Assessment |
|---|---|---|---|
| Not captured | Created evidence and test directories | PowerShell output | Establishes the intended locations for evidence and controlled activity. |
| Not captured | Checked the target volume | `Get-Volume -DriveLetter C` output | `C:` was reported as NTFS with a healthy status. |
| Not captured | Queried the USN journal configuration | `usn-journal-baseline.txt` and PowerShell output | Captured the journal ID, USN boundaries, maximum size, and allocation delta. |
| Not captured | Created `document-A.txt` | Controlled PowerShell commands | Initial content was written to the test file. |
| Not captured | Modified `document-A.txt` | Controlled PowerShell commands | Additional content was appended to the file. |
| Not captured | Copied `document-A.txt` to `document-B.txt` | Controlled PowerShell commands | Created a second file for subsequent rename and deletion. |
| Not captured | Renamed `document-B.txt` to `document-renamed.txt` | Controlled PowerShell commands | Performed a controlled file rename. |
| Not captured | Deleted `document-renamed.txt` | Controlled PowerShell commands | Removed the renamed copy. |
| Not captured | Read the USN journal | `usn-journal-output.txt` | The command redirected output to a file; the captured records still require validation. |
| Not captured | Recorded remaining-file metadata | `remaining-files.csv` | Exported metadata for files remaining in the test directory. |
| 2026-10-14 07:00:53.6788023 IST | Sysmon event document timestamp | Wazuh document `1791941454.18068897` | The supplied excerpt shows the Sysmon provider and `task: 11`, but lacks the actual event ID and target file path. Its relationship to the controlled activity is unconfirmed. |
| Not captured | Generated SHA-256 manifest | `evidence-hashes.csv` | The hashing command was executed; individual hash rows were not supplied for review. |
## USN Journal Values Recorded
| Field | Value |
|---|---|
| Journal ID | `0x01da57b6418ad7a2` |
| First USN | `0x00000018c6800000` |
| Next USN | `0x00000018c8f08f10` |
| Maximum size | 32 MB |
| Allocation delta | 8 MB |
| Supported record versions | 2 through 4 |
These are the values shown by the journal query. The exact capture time and its relation to each file operation were not established in the supplied screenshots.
## Correlation Status
The controlled operation sequence is known from the PowerShell commands, but a completed forensic correlation requires more evidence:
- Inspect the actual USN journal output and identify records matching the test file names or their file references.
- Acquire and parse MFT metadata.
- Compare USN file-reference and parent-reference values with MFT records.
- Validate timestamps against the known test window.
- Inspect the full Sysmon event to establish its actual event ID, path, and process details.
- Compare any relevant Wazuh event with the file-system artifacts.
## Evidence Integrity
A SHA-256 manifest was generated for evidence files in the evidence directory, excluding the manifest itself. The exact hash values were not shown in the supplied output.
If evidence files are changed or added, regenerate the manifest and preserve earlier manifests separately when historical comparisons are required.
## Final Assessment
The endpoint and NTFS volume were validated, controlled file operations were performed, the USN journal was queried and its output redirected to a file, and supporting metadata and a hash manifest were generated.
A complete USN/MFT timeline remains pending validation of the journal records and acquisition/parsing of MFT metadata. The displayed Sysmon event cannot yet be attributed to the test operations based on the fields supplied.
