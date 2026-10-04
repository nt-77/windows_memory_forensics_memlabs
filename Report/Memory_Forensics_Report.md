# Memory Forensics Investigation Report

## MemLabs Lab 1 — Beginner's Luck

**Investigator:** Cybersecurity Portfolio Project  
**Tool:** Volatility 3 Framework 2.28.2  
**Dataset:** MemLabs Lab 1  
**Focus:** Windows Memory Forensics

---

## 1. Executive Summary

This report documents a memory-forensics investigation of the MemLabs Lab 1 — Beginner's Luck challenge.

The investigation began with identification of the Windows memory image and process enumeration. A running `WinRAR.exe` process was identified, after which a file-object scan located `Important.rar` under the `Alissa Simpson` user profile.

The archive was recovered directly from memory using Volatility's `windows.dumpfiles` plugin. Analysis of the WinRAR process memory revealed that the archive password was based on the uppercase NTLM hash of Alissa Simpson's Windows account. The Windows account hashes were then recovered using `windows.registry.hashdump`.

The relevant NTLM hash was used to successfully decrypt the recovered archive, which contained `flag3.png`. The image contained the final challenge flag:

```text
flag{w3ll_3rd_stage_was_easy}
```

The investigation demonstrates evidence correlation across process, file, memory, and registry artifacts.

---

## 2. Evidence

### Original memory image

```text
MemoryDump_Lab1.raw
```

SHA-256:

```text
88E708413898C98351D1A5A221799FC34460DFECB20E482CA5247B0B113B8CF2
```

MD5:

```text
B9FEC1A443907D870CB32B048BDA9380
```

### Recovered archive

```text
Important.rar
```

SHA-256:

```text
6DDEB36999D7233A4E4F0E3002AE729E01263DA7A50AD3B4B4492E317CC257E9
```

### Extracted file

```text
flag3.png
```

SHA-256:

```text
A61AF048E2247E85A5CDA4229054717F15C46C1D742997F2B7EFB1B936520CDC
```

---

## 3. System Identification

The `windows.info` plugin identified the captured operating system as Windows 7 SP1, 64-bit, build 7601.17514.

Reported system time:

```text
2019-12-11 14:38:00 UTC
```

Kernel base:

```text
0xf8000261f000
```

DTB:

```text
0x187000
```

The system time is treated as metadata from the supplied memory image rather than an independently established incident timestamp.

---

## 4. Process Analysis

The `windows.pslist` plugin identified a number of processes.

Relevant processes included:

| PID | PPID | Process |
|---:|---:|---|
| 1984 | 604 | cmd.exe |
| 2424 | 604 | mspaint.exe |
| 2504 | 2680 | explorer.exe |
| 1512 | 2504 | WinRAR.exe |
| 796 | 604 | DumpIt.exe |

The WinRAR process was particularly relevant because the later file-object investigation identified an archive named `Important.rar`.

The presence of WinRAR was not interpreted as malicious by itself.

---

## 5. Network Analysis

`windows.netscan` identified multiple Windows networking artifacts.

Examples included:

- TCP 445 listening by `System`
- TCP 135 associated with a Windows service
- UDP 137 and UDP 138
- Additional listening Windows service ports

No obvious established external TCP connection was relied upon as evidence of compromise.

The network results were therefore treated as contextual artifacts rather than a definitive intrusion finding.

---

## 6. File-Object Discovery

The `windows.filescan` plugin was used to search memory for cached Windows file objects.

A relevant result was:

```text
\Users\Alissa Simpson\Documents\Important.rar
```

File-object address:

```text
0x3fb48bc0
```

This provided a concrete target for file recovery.

---

## 7. File Recovery

The file object was passed to `windows.dumpfiles`:

```text
vol -f "MemoryDump_Lab1.raw" -o "03_Exports" windows.dumpfiles --physaddr 0x3fb48bc0
```

The recovered artifact was:

```text
file.0x3fb48bc0.0xfa8001034450.DataSectionObject.Important.rar.dat
```

Size:

```text
45,056 bytes
```

### File signature

The first bytes of the recovered artifact were:

```text
52 61 72 21 1A 07 01 00
```

These bytes are consistent with the RAR 5.x archive signature.

The archive was therefore treated as a recovered RAR-format artifact.

---

## 8. Archive Analysis

The recovered archive was inspected using Python's `rarfile` package.

The archive contained:

```text
flag3.png
```

An extraction attempt demonstrated that the file was password protected.

At this stage, the investigation pivoted from archive analysis to memory analysis of the process that had opened the archive.

---

## 9. WinRAR Process-Memory Analysis

WinRAR PID 1512 was selected for deeper investigation.

The process memory was dumped using:

```text
windows.memmap --pid 1512 --dump
```

The resulting process-memory dump was:

```text
pid.1512.dmp
```

A printable-string search revealed references to:

```text
C:\Users\Alissa Simpson\Documents\Important.rar
```

It also revealed the critical password-generation clue:

```text
Password is NTLM hash(in uppercase) of Alissa's account passwd.
```

This established a relationship between the archive password and the Windows account's NTLM hash.

---

## 10. Registry and Hash Analysis

The Windows registry hives were present in memory, including the SAM and SYSTEM hives.

The `windows.registry.hashdump` plugin was used to recover account hashes.

The relevant entry was:

```text
Alissa Simpson
RID: 1003
NTLM: f4ff64c8baac57d22f22edc681055ba6
```

The clue recovered from WinRAR memory specified that the archive password should be the NTLM hash in uppercase.

Therefore the password used for the archive was:

```text
F4FF64C8BAAC57D22F22EDC681055BA6
```

This step was performed solely against the intentionally provided MemLabs challenge image.

---

## 11. Archive Decryption and Extraction

The recovered archive was successfully extracted using WinRAR with the recovered password.

The resulting directory contained:

```text
flag3.png
```

File size:

```text
46,045 bytes
```

The extracted file's SHA-256 hash was:

```text
A61AF048E2247E85A5CDA4229054717F15C46C1D742997F2B7EFB1B936520CDC
```

---

## 12. Final Finding

Opening `flag3.png` revealed:

```text
flag{w3ll_3rd_stage_was_easy}
```

This is the final challenge artifact recovered during the investigation.

---

## 13. Evidence Correlation

The investigation can be summarized as:

```text
WinRAR.exe (PID 1512)
        │
        ▼
Important.rar referenced in process/file artifacts
        │
        ▼
windows.filescan
        │
        ▼
File object 0x3fb48bc0
        │
        ▼
windows.dumpfiles
        │
        ▼
Recovered RAR archive
        │
        ▼
Archive is password protected
        │
        ▼
WinRAR process memory
        │
        ▼
Password = uppercase NTLM hash of Alissa's account
        │
        ▼
windows.registry.hashdump
        │
        ▼
Alissa Simpson NTLM hash
        │
        ▼
Archive successfully extracted
        │
        ▼
flag3.png
        │
        ▼
Final challenge flag
```

---

## 14. Timeline

| Time (UTC) | Artifact |
|---|---|
| 14:34:54 | `cmd.exe` created |
| 14:35:14 | `mspaint.exe` created |
| 14:37:14 | `explorer.exe` session created |
| 14:37:23 | `WinRAR.exe` PID 1512 created |
| 14:37:54 | `DumpIt.exe` PID 796 created |
| 14:38:00 | `windows.info` system time |

These timestamps describe observations from the memory image and should not independently be interpreted as a complete incident timeline.

---

## 15. Negative Findings and Limitations

### No definitive malicious process identified

The investigation did not classify `WinRAR.exe` as malicious simply because it was present.

### `malfind`

`windows.malfind` produced no result for WinRAR PID 1512. This is a negative finding only and does not prove that the process was completely benign.

### Network evidence

The network scan showed listening services and local networking artifacts, but did not provide sufficient evidence to claim an external connection or compromise.

### User attribution

The presence of paths under `Alissa Simpson` was used for evidence correlation within the challenge. It was not treated as independent proof of a person's actions.

### Memory limitations

Memory analysis provides a snapshot of volatile system state. Some information may have been overwritten, paged out, or unavailable at acquisition time.

---

## 16. Tools and Plugins

### Volatility 3

```text
windows.info
windows.pslist
windows.netscan
windows.filescan
windows.dumpfiles
windows.vadinfo
windows.memmap
windows.registry.hivelist
windows.registry.printkey
windows.registry.hashdump
windows.malfind
```

### Supporting tools

- Python
- Python `rarfile`
- WinRAR
- PowerShell
- SHA-256 hashing

---

## 17. Lessons Learned

1. Memory can preserve references to files even when direct access to the original file is unavailable.
2. Process memory may contain valuable application-specific clues.
3. File-object recovery can provide a bridge between process analysis and artifact extraction.
4. Registry artifacts in memory can support credential/hash-related analysis.
5. Evidence correlation is more reliable than interpreting a single artifact in isolation.
6. Negative results must be reported carefully; absence of an artifact from one plugin does not prove that the activity did not occur.
7. Hashing recovered artifacts provides a reproducible way to document evidence.

---

## 18. Conclusion

The MemLabs Lab 1 memory image provided sufficient volatile evidence to reconstruct the challenge's investigative path.

The analysis identified the operating system, enumerated processes and network artifacts, located `Important.rar`, recovered it from memory, examined the WinRAR process memory, identified the password-generation clue, recovered the relevant Windows NTLM hash, and successfully extracted `flag3.png`.

The final recovered challenge artifact was:

```text
flag{w3ll_3rd_stage_was_easy}
```

The investigation demonstrates a practical memory-forensics workflow using Volatility 3 and emphasizes the importance of correlating multiple evidence sources before reaching a conclusion.
