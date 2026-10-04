# Evidence

This directory documents the forensic artifacts recovered and generated during the memory-forensics investigation.

The original memory image and large binary artifacts are intentionally not included in this GitHub repository. They were retained locally during the investigation to preserve the original evidence and avoid uploading large forensic images.

---

## 1. Original Memory Image

**Filename:**

`MemoryDump_Lab1.raw`

**Source:** MemLabs — Lab 1: Beginner's Luck

**Role:** Primary forensic evidence used for the investigation.

### Hashes

**SHA-256:**

`88E708413898C98351D1A5A221799FC34460DFECB20E482CA5247B0B113B8CF2`

**MD5:**

`B9FEC1A443907D870CB32B048BDA9380`

The MD5 hash was verified against the published dataset value.

The original memory image was preserved separately and was not modified during the investigation.

---

## 2. Recovered Archive — Important.rar

During the investigation, Volatility 3's `windows.filescan` plugin identified a cached file object corresponding to:

```text
C:\Users\Alissa Simpson\Documents\Important.rar