# Project 2 — Windows Memory Forensics

## MemLabs Lab 1 — Beginner's Luck

**Project type:** Digital Forensics / Memory Forensics  
**Framework:** Volatility 3  
**Operating system in memory:** Windows 7 SP1 x64  
**Dataset:** MemLabs Lab 1 — Beginner's Luck

---

## 1. Project Overview

This project demonstrates a practical Windows memory-forensics investigation using **Volatility 3**.

The investigation starts with a raw Windows memory image and follows a forensic workflow to identify relevant processes, investigate application memory, recover a file object, analyze Windows registry artifacts, recover an NTLM hash, decrypt a protected archive, and recover the final challenge artifact.

The investigation ultimately recovered:

```text
flag3.png
```

containing the challenge flag:

```text
flag{w3ll_3rd_stage_was_easy}
```

The project focuses on **evidence correlation** rather than treating a single artifact as conclusive evidence.

---

## 2. Investigation Objectives

The main objectives were to:

- Identify the operating system and system context.
- Enumerate running processes.
- Analyze process relationships and command lines.
- Examine network artifacts.
- Inspect loaded DLLs and virtual memory regions.
- Identify and recover a relevant file object from memory.
- Analyze the recovered archive.
- Investigate the memory of the associated WinRAR process.
- Identify the archive password-generation mechanism.
- Analyze Windows registry hives.
- Recover the NTLM hash for the relevant Windows account.
- Decrypt the recovered archive.
- Recover and hash the final artifact.
- Document the complete forensic workflow.

---

## 3. Environment

### Host

| Component | Specification |
|---|---|
| Host OS | Windows 11 Enterprise 64-bit |
| RAM | 20 GB |
| Virtualization | VMware Workstation |

### Investigation VM

| Component | Specification |
|---|---|
| RAM | 4 GB |
| Processors | 2 |
| Disk | 40 GB |

### Tools

- Volatility 3 Framework 2.28.2
- Python 3.13.15
- Python `rarfile` 4.5
- WinRAR
- PowerShell

---

## 4. Evidence

The original memory image was:

```text
MemoryDump_Lab1.raw
```

The original evidence was kept separately from working files.

### Original Evidence Hashes

**SHA-256**

```text
88E708413898C98351D1A5A221799FC34460DFECB20E482CA5247B0B113B8CF2
```

**MD5**

```text
B9FEC1A443907D870CB32B048BDA9380
```

The MD5 value matched the published dataset MD5.

> The original memory image is not included in this repository. The repository documents the analysis and selected derived artifacts only.

---

# 5. Investigation Workflow

```text
Memory Image
     │
     ▼
System Identification
     │
     ▼
Process Enumeration
     │
     ▼
Process Tree / Command Line
     │
     ▼
Identify WinRAR PID 1512
     │
     ▼
Investigate WinRAR Memory
     │
     ├──────────────► Password rule discovered
     │
     ▼
Find Important.rar in memory
     │
     ▼
Recover Important.rar
     │
     ▼
Analyze RAR archive
     │
     ▼
Identify Alissa Simpson account
     │
     ▼
Recover NTLM hash
     │
     ▼
Use uppercase NTLM hash as password
     │
     ▼
Extract flag3.png
     │
     ▼
Validate final artifact
```

---

# 6. Key Findings

### WinRAR Process

A running WinRAR process was identified:

```text
PID: 1512
```

Its command line referenced:

```text
C:\Users\Alissa Simpson\Documents\Important.rar
```

### Recovered Archive

`Important.rar` was located in memory and recovered using:

```text
windows.filescan
windows.dumpfiles
```

Recovered artifact:

```text
file.0x3fb48bc0.0xfa8001034450.DataSectionObject.Important.rar.dat
```

SHA-256:

```text
6DDEB36999D7233A4E4F0E3002AE729E01263DA7A50AD3B4B4492E317CC257E9
```

The artifact had a RAR 5.x signature and contained:

```text
flag3.png
```

### Password Rule

Analysis of the WinRAR process memory revealed:

```text
Password is NTLM hash (in uppercase) of Alissa's account passwd.
```

### Windows Account

The relevant account was:

```text
Alissa Simpson
```

RID:

```text
1003
```

### NTLM Hash

Recovered using:

```text
windows.registry.hashdump
```

Hash:

```text
f4ff64c8baac57d22f22edc681055ba6
```

The uppercase value was used as the archive password:

```text
F4FF64C8BAAC57D22F22EDC681055BA6
```

### Final Artifact

Recovered:

```text
flag3.png
```

SHA-256:

```text
A61AF048E2247E85A5CDA4229054717F15C46C1D742997F2B7EFB1B936520CDC
```

Final flag:

```text
flag{w3ll_3rd_stage_was_easy}
```

---

# 7. Volatility Plugins Used

| Plugin | Purpose |
|---|---|
| `windows.info` | Identify Windows system information |
| `windows.pslist` | Enumerate processes |
| `windows.pstree` | Examine process relationships |
| `windows.cmdline` | Recover process command lines |
| `windows.netscan` | Analyze network artifacts |
| `windows.dlllist` | Examine loaded DLLs |
| `windows.malfind` | Check for suspicious memory regions |
| `windows.vadinfo` | Examine process virtual memory |
| `windows.memmap` | Dump process memory |
| `windows.filescan` | Locate file objects in memory |
| `windows.dumpfiles` | Recover cached file data |
| `windows.registry.hivelist` | Identify registry hives |
| `windows.registry.printkey` | Examine registry keys |
| `windows.registry.hashdump` | Recover Windows account hashes |

---

# 8. Repository Structure

```text
Project2_Memory_Forensics/
│
├── README.md
│
├── Report/
│   └── Memory_Forensics_Report.md
│
├── Screenshots/
│   └── README.md
│
└── Exports/
    └── README.md
```

The repository intentionally does **not** include the original memory image.

Large forensic evidence files should remain outside the Git repository unless their redistribution is explicitly permitted.

---

# 9. Ethical and Forensic Considerations

This project was performed against a dedicated educational memory-forensics challenge dataset.

Recovered credentials and hashes are documented strictly as part of the forensic analysis of the provided challenge image.

The original memory image should not be redistributed unless permitted by the dataset's licensing terms.

No brute-force password attack was required. The password was derived directly from the forensic evidence present in the memory image.

---

# 10. Skills Demonstrated

This project demonstrates practical experience with:

- Digital forensics
- Windows memory forensics
- Volatility 3
- Process analysis
- Process-tree analysis
- Command-line analysis
- Network artifact analysis
- Virtual memory analysis
- Process memory dumping
- Memory string analysis
- Windows file-object recovery
- Registry analysis
- SAM analysis
- NTLM hash recovery
- Archive analysis
- Evidence hashing
- Evidence correlation
- Forensic documentation

---

## 11. Final Result

The investigation successfully recovered the final challenge artifact:

```text
flag3.png
```

SHA-256:

```text
A61AF048E2247E85A5CDA4229054717F15C46C1D742997F2B7EFB1B936520CDC
```

The artifact contained:

```text
flag{w3ll_3rd_stage_was_easy}
```

---

## Author

**[Your Name]**

Master's in Information Security  
Digital Forensics / DFIR Portfolio Project
