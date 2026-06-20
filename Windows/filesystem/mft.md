## What is the MFT?

The **Master File Table (MFT)** is the core database of the **NTFS (New Technology File System)** used by Windows.

Every file and directory stored on an NTFS volume has at least one corresponding entry in the MFT.

Think of the MFT as a giant index that stores metadata about files rather than the actual file contents themselves.

The MFT records information such as:

- File name
- File size
- Timestamps
- Permissions
- Owner information
- File location on disk
- Attributes (Hidden, System, Archive, etc.)

Without the MFT, Windows would not be able to locate or manage files on the disk.

---

## Importance of MFT

The MFT is one of the most valuable artifacts in Windows Digital Forensics because it provides evidence of:

- File creation
- File modification
- File deletion
- File renaming
- User activity
- Malware activity
- Data hiding attempts

Even after a file is deleted, remnants of its MFT entry may still exist, making it possible to recover forensic evidence.

---

## MFT Record

Each file or directory has a corresponding **MFT Record**.

### Key Facts

| Property | Description |
|-----------|------------|
| Record Size | Usually 1024 bytes |
| Record Number | Unique identifier for each record |
| Signature | Begins with `FILE` |
| Contains | Metadata attributes |
| Reused | Deleted records may later be reused |

Example:

```text
Record #12345
|
+-- File Name: malware.exe
+-- Size: 256 KB
+-- Created: 2025-06-01
+-- Modified: 2025-06-02
+-- Owner: User
```

---

# Important MFT Attributes

An MFT record consists of several attributes.

## 1. $STANDARD_INFORMATION

Contains basic metadata about the file.

### Stores

- Created Time
- Modified Time
- MFT Modified Time
- Last Accessed Time
- File Attributes

These timestamps are often referred to as:

```text
MACB Times

M - Modified
A - Accessed
C - Changed (MFT Entry Modified)
B - Born (Created)
```

### Forensic Value

This is usually the first place investigators look when building a file activity timeline.

---

## 2. $FILE_NAME

Contains filename-related information.

### Stores

- File Name
- Parent Directory
- Additional Timestamps
- Allocated Size
- Real Size

### Forensic Value

A file has timestamps in both:

- `$STANDARD_INFORMATION`
- `$FILE_NAME`

Comparing these timestamps can reveal:

- Timestamp manipulation
- File copying
- Anti-forensic activity

---

## 3. $DATA

Contains the actual file content.

Example:

```text
document.txt
```

The text inside the file resides in the `$DATA` attribute.

### Alternate Data Streams (ADS)

NTFS also supports **Alternate Data Streams (ADS)**.

Example:

```powershell
notepad.exe:hidden.txt
```

Attackers sometimes abuse ADS to hide data from casual inspection.

---

## 4. $LOGGED_UTILITY_STREAM

Used by **Transactional NTFS (TxF)**.

Stores information related to transaction processing and file state changes.

### Forensic Value

May provide evidence of:

- File operations
- System transactions
- Historical changes

---

## 5. $BITMAP

Tracks which clusters belonging to the file are allocated.

### Forensic Value

Useful when analyzing:

- Fragmented files
- File allocation behavior
- Storage usage

---

## 6. $SECURITY_DESCRIPTOR

Contains security information.

### Stores

- Owner SID
- Group SID
- Access Control Lists (ACLs)

### Forensic Value

Helps identify:

- File ownership
- User permissions
- Unauthorized access

---

## 7. $VOLUME_INFORMATION

Present in special system records.

Contains:

- NTFS version
- Volume flags
- Volume metadata

---

## 8. $INDEX_ROOT and $INDEX_ALLOCATION

Primarily used by directories.

These attributes maintain directory indexes and allow Windows to quickly locate files.

### Forensic Value

Can reveal:

- Previously existing files
- Directory contents
- File movement evidence

---

# MFT Timestamps

One of the most important forensic uses of the MFT is timeline analysis.

Each file may contain multiple timestamps.

| Timestamp | Description |
|------------|------------|
| Created | When file was created |
| Modified | When content changed |
| Accessed | When file was opened/accessed |
| MFT Changed | When metadata changed |

### Example Timeline

```text
09:00 - malware.exe created
09:02 - malware.exe executed
09:05 - malware.exe modified
09:10 - malware.exe deleted
```

The MFT can help reconstruct this sequence of events.

---

# Significance of MFT in Digital Forensics

## 1. File Activity Reconstruction

Investigators can determine:

- What happened
- When it happened
- Which file was involved

---

## 2. Deleted File Analysis

Deleted files often leave behind MFT records.

This helps recover:

- File names
- Paths
- Timestamps
- Metadata

Even when the file content itself is gone.

---

## 3. Malware Investigation

The MFT can reveal:

- Malware drop locations
- Suspicious executables
- Unexpected file creations
- Persistence-related files

---

## 4. Timeline Creation

MFT timestamps are frequently combined with:

- Event Logs
- Registry Artifacts
- Prefetch Files
- Browser History

to build a complete attack timeline.

---

## 5. Detecting Anti-Forensics

Attackers may attempt to:

- Delete files
- Rename files
- Modify timestamps

Comparing timestamps across MFT attributes may reveal such attempts.

---

# Useful MFT Analysis Tools

## MFTECmd

Created by Eric Zimmerman.

### Features

- Parses MFT records
- Exports CSV files
- Supports timeline generation
- Fast and widely used in DFIR

Example:

```powershell
MFTECmd.exe -f $MFT --csv output
```

---

## Timeline Explorer

Used to analyze CSV output from MFTECmd.

### Features

- Sorting
- Filtering
- Timeline reconstruction
- Large dataset analysis

Typical workflow:

```text
$MFT
  ↓
MFTECmd
  ↓
CSV Output
  ↓
Timeline Explorer
```

---

## MFTExplorer

GUI-based MFT analysis tool.

### Features

- Browse MFT records
- View timestamps
- Examine attributes
- Analyze deleted entries

Useful for learning and quick investigations.

---

# Common DFIR Workflow

```text
Acquire Disk Image
        ↓
Extract $MFT
        ↓
Parse with MFTECmd
        ↓
Generate CSV
        ↓
Analyze in Timeline Explorer
        ↓
Build Incident Timeline
```

---

# Real-World Investigation Example

### Scenario

A user claims they never downloaded `payload.exe`.

### Investigation

1. Extract the `$MFT`.
2. Locate the MFT record for `payload.exe`.
3. Review MACB timestamps.
4. Identify the parent directory.
5. Correlate timestamps with:
   - Event Logs
   - Browser History
   - Prefetch
   - Registry Artifacts

### Possible Findings

- File created from browser download.
- File executed shortly afterward.
- File deleted after execution.
- Evidence of malware activity.

---

## Exam refresher

- Every file on NTFS has an MFT record.
- MFT is the heart of NTFS.
- Record signature = `FILE`
- Typical record size = 1024 bytes.
- Deleted files may still leave MFT entries.
- `$STANDARD_INFORMATION` and `$FILE_NAME` both contain timestamps.
- MACB timestamps are heavily used in DFIR investigations.
- MFTECmd is one of the most commonly used tools for MFT parsing.
- Comparing timestamp attributes can reveal anti-forensic activity.
- MFT is often the starting point for Windows forensic investigations.
