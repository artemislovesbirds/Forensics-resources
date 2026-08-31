# The world of deleted files
## A presentation on how to work with images/partitions and file recovery
### By Mia

---
# Introduction

## Why disk image forensics is interesting
1. Professional and academic relevance
  - Blue team: Investigative work (incident response, police investigations)
  - Red team: anti-forensics
  - Open problems [1]:
    - Standard datasets, SOP (organizational change), anti-forensics 
---
# Introduction

## Why disk image forensics is interesting
1. Professional and academic relevance
  - Blue team: Investigative work (incident response, police investigations)
  - Red team: anti-forensics
  - Open problems [1]:
    - Standard datasets, SOP (organizational change), anti-forensics 
2. It is fun
  - You will feel like a detective
---
# Introduction

## Why disk image forensics is interesting
1. Professional and academic relevance
  - Blue team: Investigative work (incident response, police investigations)
  - Red team: anti-forensics
  - Open problems [1]:
    - Standard datasets, SOP (organizational change), anti-forensics 
2. It is fun
  - You will feel like a detective
3. Useful
  - Commandline interface (grep, file analysis)
  - Skills are transferable: reverse engineering
---
# What you will (hopefully) learn
1. The digital forensics and CTF forensics procedure
2. Basic information about file systems
3. How to access disk images while ensuring the integrity of the data
4. How to recover deleted files
5. How to analyze them
6. If we have time: anti-forensics against file recovery
---
# The plan
1. Basic practices
2. File system basics
3. Working with images safely
4. Types of forensics analysis
5. Inspecting the image
6. Analysing the data
7. Walkthrough of a challenge
8. Anti-forensics (if we have time)
9. Resources and recommendations

---
# Forensic procedure
## Digital forensics Lite

Traditional digital forensics process:
1. Identification
2. Preservation - hash, copy, never touch the original
3. Collection - physical vs logical acquisition
4. Examination - inspect (and verify)
5. Analysis - construct the narrative
6. Reporting

Chain of custody, care and control
1. Log relevant information
2. Control access
3. Perform analysis on mirror-image copies
---
# Forensic procedure

CTF digital forensics process (according to me):
1. Preservation - Main goal: avoid corrupting the data
2. Examination - inspect (and find) files of interest
3. Analysis - interpret the data
No identification or collection needed (the files are given to you) 
No reporting needed (except for handing in the flag)
No chain of custody
---
# File system basics
### Mostly based on Lavarian's post [2]
1. For our purposes: files are connected data
2. A directory (also a file) organizes groups of files
3. A file system defines how files are named, stored, and retrieved from a storage device
   - For our purposes, this broad definition is enough

Storage devices (hereafter disk) contain information, sometimes including an OS, file system, files etc.
A disk image is a *snapshot* of a disk
A disk is usually partitioned i.e. split into different sections. 
1. On a computer there is usually one partition for the OS (and related files), one for user files, and one for swapping (not important for this presentation)
---
# File system basics
## Index Node (inodes)
1. Special data structure
2. Determine number of files on a storage device
3. Contains metadata on files
   - Ownership
   - Permissions
   - Size
   - Timestamps
   - Block pointers (i.e. location)
     - Data on a storage device is sectioned into blocks (e.g. 4096 bytes per block, `stat dir` in linux to see what your system uses)
     - Indirect single, double and triple blocks (might delete unless used as an example later)
---
# File system basics
## Deleting files


# File system basics
## Journaling


# Resources
[1] Arshad, Humaira & Jantan, Aman & Abiodun, Oludare. (2018). Digital Forensics: Review of Issues in Scientific Validation of Digital Evidence. Journal of Information Processing Systems. 14. 346 ~ 376. 10.3745/JIPS.03.0095.
[2] https://www.freecodecamp.org/news/file-systems-architecture-explained
