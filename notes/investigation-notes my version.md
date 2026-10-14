# Lab 106 — USN + MFT Timeline Investigation Notes
## 1. System and Volume Validation
The PowerShell output reported the following:
| Field | Observed value |
|---|---|
| Drive letter | `C:` |
| File system | `NTFS` |
| Health status | `Healthy` |
| Reported volume size | `103784960000` bytes |
| Test directory | `C:\USNMFTTimelineLab\TestFiles` |
| Evidence directory | `C:\USNMFTTimelineLab\Evidence` |
The volume check established that the target volume uses NTFS, which is required for this USN/MFT investigation.
## 2. USN Journal Configuration
The `fsutil usn queryjournal C:` command reported the following values:
| Field | Observed value |
|---|---|
| USN Journal ID | `0x01da57b6418ad7a2` |
| First USN | `0x00000018c6800000` |
| Next USN | `0x00000018c8f08f10` |
| Lowest Valid USN | `0x0000000000000000` |
| Maximum USN | `0x00000fffffff0000` |
| Maximum size | 32 MB |
| Allocation delta | 8 MB |
| Minimum supported record version | 2 |
| Maximum supported record version | 4 |
| Write range tracking | Disabled |
The output was saved to `usn-journal-baseline.txt`.
These values describe the journal configuration at the time of the query. The screenshots do not establish the exact relationship between the query timestamp and the individual file operations. Therefore, these values should not automatically be described as a verified pre-test baseline unless the collection order is confirmed.
## 3. Controlled File-System Activity
The test script defined these paths:
- `document-A.txt`
- `document-B.txt`
- `document-renamed.txt`
The operations were:
1. Write `Initial content` to `document-A.txt`.
2. Append `Additional content for timeline testing`.
3. Copy the file to `document-B.txt`.
4. Rename the copy to `document-renamed.txt`.
5. Delete `document-renamed.txt`.
The commands completed and returned to the PowerShell prompt without a visible error in the supplied screenshots.
The script does not remove `document-A.txt`. Based on the recorded commands, it should remain in the test directory. Verify the actual result in `remaining-files.csv` before documenting the final state as confirmed.
The screenshots do not provide exact timestamps for each operation. These must be recovered from any preserved action log, USN records, or other available evidence rather than estimated from the order of the screenshots.
## 4. USN Journal Collection
The following command was executed:

`fsutil usn readjournal C: | Out-File "$LabPath\usn-journal-output.txt" -Encoding utf8`

The command redirected the journal output to `usn-journal-output.txt`. The screenshot shows the PowerShell prompt returning without a visible error, but the file contents and record count were not supplied.
The next validation steps are:
- Confirm that the output file exists and is non-empty.
- Inspect its first and last records.
- Search for the test file names.
- Identify the USN reason flags and file-reference values associated with candidate records.
- Compare the record timestamps with the known operation window.
- Confirm that records are from the expected volume and journal.
An output file existing on disk does not, by itself, prove that the expected test records were successfully captured.
## 5. MFT Analysis Status
The supplied evidence does not show a raw MFT acquisition or a parsed MFT export.
Consequently, this investigation cannot yet report verified MFT file-reference numbers, parent references, or correlations between MFT records and USN records.
To complete this portion, acquire or parse the appropriate NTFS MFT metadata using a suitable forensic tool. Document the tool and version, collection method, source volume, output format, and any limitations.
## 6. Sysmon and Wazuh Telemetry
The supplied Wazuh event excerpt includes:
| Field | Observed value |
|---|---|
| Provider name | `Microsoft-Windows-Sysmon` |
| Provider GUID | `{5770385f-c22a-43e0-bf4c-06f5698ffbd9}` |
| System time | `2026-10-14T01:30:53.6788023Z` |
| Task | `11` |
| Thread ID | `7356` |
| Version | `2` |
| Decoder | `windows_eventchannel` |
| Event document ID | `1791941454.18068897` |
The system time converts to `2026-10-14 07:00:53.6788023` in India Standard Time (UTC+05:30).
The supplied excerpt does not show `data.win.system.eventID`, `data.win.eventdata.targetFilename`, or the process details required to establish whether this event is related to the test directory.
The field `data.win.system.task` must not be treated as the Sysmon Event ID. Inspect the full Wazuh event JSON before classifying this record as a file-creation event or attributing it to the controlled activity.
## 7. Evidence Hashing
The following hashing command was executed:
`Get-ChildItem -LiteralPath $LabPath -File | Where-Object { $_.FullName -ne $HashFile } | Get-FileHash -Algorithm SHA256 | Select-Object Path, Algorithm, Hash | Export-Csv -LiteralPath $HashFile -NoTypeInformation`
The output was written to `evidence-hashes.csv`. The screenshots do not include the resulting manifest rows, so the individual hashes and the completeness of the manifest have not been independently reviewed here.
The manifest should be re-generated after any evidence file is modified. Preserve prior manifests separately if a historical record of artifact changes is required.
## 8. Preliminary Findings
### Confirmed by the supplied output
- The `C:` volume uses NTFS and was reported as healthy.
- The USN journal configuration was queried and its values were displayed.
- The controlled file-operation commands were executed.
- The journal read command redirected its output to a file.
- A remaining-file metadata export command was executed.
- A SHA-256 manifest generation command was executed.
- A Wazuh document containing Sysmon provider metadata was available.
### Not yet established
- Whether `usn-journal-output.txt` contains the complete expected set of test records.
- Whether individual USN records can be correlated to each controlled file operation.
- The MFT file-reference numbers and parent references for the test files.
- Whether the displayed Sysmon event references the test directory.
- Whether every evidence file is represented in a reviewed SHA-256 manifest.
- A complete, independently validated USN/MFT timeline.
