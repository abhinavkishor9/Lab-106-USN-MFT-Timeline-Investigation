# Lab 106 — USN + MFT Timeline Investigation Troubleshooting Notes
## 1. `fsutil usn readjournal C: ?` Returned an Invalid Parameter
### Observed behavior
The command returned an invalid-parameter message and printed the supported `fsutil usn readJournal` syntax.
### Explanation
The literal `?` is not a supported help parameter for this command on the displayed Windows version. The command output itself provided the supported options.
### Correct usage
To query journal configuration:
```powershell
fsutil usn queryjournal C:
```
To read the journal:
```powershell
fsutil usn readjournal C:
```
To request CSV-formatted output:
```powershell
fsutil usn readjournal C: csv
```
To start reading from a specified USN:
```powershell
fsutil usn readjournal C: startUsn=0x00000018c6800000
```
Use the value appropriate to the collection plan. The first USN shown by `queryjournal` is not a guarantee that every record since that point remains available; verify the actual returned output.
## 2. Verify the Journal Output File
### Potential issue
The `fsutil` command redirected output to `usn-journal-output.txt`, but the supplied screenshots do not show the file contents or its size.
### Validation
```powershell
$LabPath = "C:\USNMFTTimelineLab\Evidence"
$UsnFile = Join-Path $LabPath "usn-journal-output.txt"

if (-not (Test-Path -LiteralPath $UsnFile -PathType Leaf)) {
    throw "USN output file was not found: $UsnFile"
}

Get-Item -LiteralPath $UsnFile |
    Select-Object FullName, Length, LastWriteTime

Get-Content -LiteralPath $UsnFile -TotalCount 20
```
If the file is empty or does not contain readable records, collect it again using the supported syntax and inspect the exit status and output.
For CSV output:
```powershell
$LabPath = "C:\USNMFTTimelineLab\Evidence"
$UsnFile = Join-Path $LabPath "usn-journal-output.csv"

& fsutil.exe usn readjournal C: csv |
    Out-File -LiteralPath $UsnFile -Encoding utf8

$LASTEXITCODE
Get-Item -LiteralPath $UsnFile |
    Select-Object FullName, Length, LastWriteTime
```
The journal may be large. Use an appropriate collection window or start USN when justified, and preserve enough context to explain the collection boundaries.
## 3. Test File Names Do Not Appear in a Journal Search
### Possible causes
- The output file contains a different range or format than expected.
- The file names appear in a different representation or require parsing.
- Relevant journal records are no longer available.
- The search is being performed against the wrong journal output file.
- The query or parser does not correctly interpret the record format.
### Troubleshooting
First search the raw output for the expected names:
```powershell
$LabPath = "C:\USNMFTimelineLab\Evidence"
$UsnFile = Join-Path $LabPath "usn-journal-output.txt"

Select-String -LiteralPath $UsnFile `
    -Pattern "document-A.txt|document-B.txt|document-renamed.txt"
```
If no matches appear, confirm the file format and inspect representative records. A raw text search may not be sufficient for all output formats; parsing the records with an appropriate forensic tool may be necessary.
Do not conclude that a file operation did not occur merely because a text search returned no match.
## 4. Validate `remaining-files.csv`
The script exports metadata for files that remain in the test directory:
```powershell
$LabPath = "C:\USNMFTTimelineLab\Evidence"
$RemainingFile = Join-Path $LabPath "remaining-files.csv"

Import-Csv -LiteralPath $RemainingFile |
    Format-Table FullName, Length, CreationTimeUtc, LastWriteTimeUtc,
                 LastAccessTimeUtc, Attributes -AutoSize
```
Based on the supplied test commands, `document-A.txt` should remain because the script deletes only `document-renamed.txt`. Inspect the CSV to confirm the actual final state.
## 5. Validate the Wazuh Sysmon Event
### Observed behavior
The displayed event contains the Sysmon provider name and `data.win.system.task: 11`. The supplied excerpt does not show the actual event ID or target file name.
### Troubleshooting
In Wazuh Discover, first use the known-working agent filter:
```text
agent.id:"001"
```
Then narrow the results:
```text
agent.id:"001" AND data.win.system.providerName:"Microsoft-Windows-Sysmon"
```
For Sysmon file creation events:
```text
agent.id:"001" AND data.win.system.eventID:11
```
For creation events involving the lab directory, try:
```text
agent.id:"001" AND data.win.system.eventID:11 AND data.win.eventdata.targetFilename:*USNMFTTimelineLab*
```
If the combined query fails or returns no results, inspect a known event's complete JSON and confirm the indexed field names. Keep the time range around the actual lab activity.
**Important:** `data.win.system.task` is not a substitute for `data.win.system.eventID`. Do not infer the event type from the task field alone.
## 6. MFT Artifact Has Not Been Collected
The built-in commands shown so far query the USN journal and list normal file metadata. They do not provide a complete parsed MFT dataset.
To continue:
- Choose a suitable, trusted NTFS forensic acquisition or parsing tool.
- Record the tool name and version.
- Document whether it acquires raw MFT metadata or parses an existing acquisition.
- Preserve the original output and hash it before conducting transformations where practical.
- Record any access restrictions, parsing errors, or fields the tool cannot interpret.
Do not claim that an MFT-to-USN correlation has been completed until the MFT records and corresponding USN records have been examined.
## 7. Hashing Errors or Missing Hashes
Validate the evidence directory before hashing:
```powershell
$LabPath = "C:\USNMFTTimelineLab\Evidence"

if (-not (Test-Path -LiteralPath $LabPath -PathType Container)) {
    throw "Evidence directory not found: $LabPath"
}

Get-ChildItem -LiteralPath $LabPath -File |
    Select-Object Name, Length, LastWriteTime
```
Regenerate the manifest after evidence files are corrected or added:
```powershell
$LabPath = "C:\USNMFTTimelineLab\Evidence"
$HashFile = Join-Path $LabPath "evidence-hashes.csv"

Get-ChildItem -LiteralPath $LabPath -File |
    Where-Object { $_.FullName -ne $HashFile } |
    Get-FileHash -Algorithm SHA256 |
    Select-Object Path, Algorithm, Hash |
    Export-Csv -LiteralPath $HashFile -NoTypeInformation

Import-Csv -LiteralPath $HashFile |
    Format-Table -AutoSize
```
This overwrites the current manifest. Preserve an earlier version separately when historical integrity records are needed.
## 8. Distinguish File-System Artifacts
- **USN journal:** Records changes to files and directories, subject to journal retention and collection limits.
- **MFT:** Provides file-system records and metadata that can help identify file references and relationships.
- **Directory listing:** Shows files currently visible in that directory; it does not recreate a complete history of deleted or renamed files.
- **Sysmon/Wazuh:** Provides configured event telemetry that may help correlate operations with processes, but does not automatically provide a complete USN journal or MFT.
Do not treat these sources as interchangeable. Use each artifact for what it actually records.
## 9. Final Validation Checklist
- [ ] Confirm the target volume is NTFS.
- [ ] Preserve the USN journal configuration.
- [ ] Verify that the journal output file exists and contains usable records.
- [ ] Search and parse records associated with the controlled file operations.
- [ ] Confirm the final state of the test directory.
- [ ] Inspect the full Sysmon event JSON and identify the actual event ID.
- [ ] Acquire and parse MFT metadata.
- [ ] Correlate file references, timestamps, names, and USN reason flags.
- [ ] Generate and inspect a SHA-256 manifest.
- [ ] Document missing evidence and unresolved correlation questions.
