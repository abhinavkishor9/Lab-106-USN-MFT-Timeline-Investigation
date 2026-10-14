# Timeline Investigation
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
