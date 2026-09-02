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
2. Brief recap about file systems (could be its own workshop, this is the sweet and **short** version)
3. File deletion on different file systems
4. How to access disk images while ensuring the integrity of the data
5. How to recover deleted files
6. How to analyze them
7. If we have time: anti-forensics against file recovery
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
No identification or collection needed (the files are given to you) <br>
No reporting needed (except for handing in the flag) <br>
No chain of custody
---
# File system basics
### Mostly based on Lavarian's post [2]
1. For our purposes: files are connected data
2. A directory (also a file) organizes groups of files
3. A file system defines how files are named, stored, and retrieved from a storage device
   - For our purposes, this broad definition is enough

Storage devices (hereafter disk) contain information, sometimes including an OS, file system, files etc. <br>
A disk image is a *snapshot* of a disk <br>
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
     - Data on a storage device is sectioned into blocks
     - Check block size with `stat dirname` in linux or mac terminal, `fsutil fsinfo ntfsInfo C:` in Windows powershell (blocks are named clusters in Windows)
     - Indirect single, double and triple blocks (might delete unless used as an example later)
---
# File deletion
What happens when a file is deleted?
1. It depends (on the file system)
2. When a file is deleted in ext4 the inode reference is removed along with the metadata (e.g. the block pointer)
3. When a file is deleted in NTFS:
   - The Master File Table (MFT) entry is set to free
     - It still contains information about the file until it is overwritten 
   - The file is put into the recycle bin
     - Parent directory is set to $Recycle_Bin
     - File renamed to $R\[randomSixCharacters\]
     - A file with the same name as the deleted file except starting with $I is added containing metadata about the original file e.g. original directory
       - An I$ file is added each time the file is deleted
       - Interesting from a forensics standpoint even if the R$ file has been deleted
4. When a file is deleted in HFS+
   - The catalog record (MacOS' Inode equivalent) in the Catalog File is removed 
   - APS complicates recovery
     - TRIM -> asynchronous deletion on the SSD (look it up)
     - Native encryption (crypto-erase) -> key is gone == data (practically) not recoverable
     - Snapshots -> (look it up)
     - Copy-on-write -> Data is not modified in place, instead in an Object Map (look that up)
6. For all of these file systems, the data still sits on it's block even if there is no pointer to it
   - It can, however, be overwritten
   - Crypto-erase -> data not recoverable
7. Bonus:
   - StegFS
   - exFAT (read the paper because this is fun)

# Safe disk analysis
Ideally copy and hash e.g.: <br>
`dd if=/dev/SOURCE of=/dev/DESTINATION` <br>
`md5sum disk.img` <br>
1. Makes it easier to ensure you are not working on a faulty copy
   - Flag might have been deleted (example on next slide)

CTF is low stakes, you can always download the file again

# Journaling




# Resources
[1] Arshad, Humaira & Jantan, Aman & Abiodun, Oludare. (2018). Digital Forensics: Review of Issues in Scientific Validation of Digital Evidence. Journal of Information Processing Systems. 14. 346 ~ 376. 10.3745/JIPS.03.0095.
[2] https://www.freecodecamp.org/news/file-systems-architecture-explained
