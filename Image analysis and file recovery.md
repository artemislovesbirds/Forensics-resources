# Disk image forensics for CTFs
## By Mia

---
# whoami
My name is Mia <br>
I study software design and I am writing my thesis <br>
I made the 'Bird Lover' and 'Something sounds off' challenges <br>
  - And no one has solved them yet! Go and do them right after this
  - Fun and bird related forensics (Steganography)
  - Not related to disk forensics
<br>
In retrospect I should have done a workshop on Steganography so you could actually do them (oh well)

---
# Brief introduction to the topic
What is digital forensics?
- The analysis of digital media to detect forgery or manipulation (Wiktionary)
- Digital forensics is the process of collecting and analyzing digital evidence in a way that maintains its integrity and admissibility in court. (IBM)

What is forensics in a CTF context? <br>
Challenges that deal with topics like:
- Examining file metadata, finding hidden files in directories, changing image scaling to find the flag (common tricks you might see)
- Finding hidden data within files (Steganography)
  - Includes examining hex values within images, finding hidden messages in pixels, finding hidden data in audio files, extracting embedded files and much more
- Examining network data (Network forensics)
  - Identifying suspicious traffic, finding leaked data in packets and more
- Examining disk images (Disk forensics, our focus today)
  - Examining deleted files and deleted data in the slack space, disk image anti-forensics, detecting timeline tampering (will not be covered today) and more
- Forensics as a whole has more topics within incident response, malware analysis, forgery detection, anti-forensics etc. these are just the types of challenges I have seen the most
- Anti-forensics: avoiding detection
---
# Why disk image forensics is interesting
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
# What I need from you
  - I assume you know how a CTF works (if not raise your hand)
  - Please install sleuthkit now if you want to follow along and haven't already installed it
    - on linux: sudo apt install sleuthkit
    - Mac and Windows: download available online, just search for "sleuthkit download"
  - If you want to do the harder challenges on this topic install autopsy (comes with Kali linux)
  - Please ask questions if you are confused (about forensics, not in general)
  - Please clap at the end and tell me how good it was
---
# What you will (hopefully) learn
1. The digital forensics and CTF forensics procedure
2. Brief recap about file systems (could be its own workshop, this is the sweet and ***short*** version)
3. File deletion on different file systems
4. How to access disk images while ensuring the integrity of the data
5. How to recover deleted files
6. How to analyze them
7. If we have time: anti-forensics against file recovery and more
There will be a walkthrough of two challenges from Cylab (formerly PicoCTF)[2]
---
# Forensic procedure
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
 - Some challenges detect bad practices and delete the flag (more on this later) 
2. Examination - inspect and find files of interest (no need to verify)
3. Analysis - interpret the data (find the flag)
No identification or collection needed (the files are given to you) <br>
No reporting needed (except for handing in the flag) <br>
No chain of custody
---
# File system basics
### Mostly based on Lavarian's post [3]
1. For our purposes: files are connected data
2. A directory (also a file) organizes groups of files
3. A file system defines how files are named, stored, and retrieved from a storage device
   - For our purposes, this broad definition is enough

Storage devices (hereafter disk) contain information, sometimes including an Operating System, file system, files etc. <br>
A disk image is a *snapshot* of a disk <br>
A disk is usually partitioned i.e. split into different sections.
  - On a computer there is usually one partition for the OS (and related files), one for user files, and one for swapping
  - We care about user files (generally) 
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
     - Check block size with `stat -f filename` in linux or mac terminal, `fsutil fsinfo ntfsInfo C:` in Windows powershell (blocks are named clusters in Windows)
     - Indirect single, double and triple blocks (might delete unless used as an example later)
---
# File deletion
What happens when a file is deleted?
1. It depends (on the file system)
2. When a file is deleted in ext4 (Linux) the inode reference is removed along with the metadata (e.g. the block pointer)
3. When a file is deleted in the NT File System (NTFS) (Windows):
   - The Master File Table (MFT) entry is set to free
     - It still contains information about the file until it is overwritten 
   - The file is put into the recycle bin
     - Parent directory is set to $Recycle_Bin
     - File renamed to $R\[randomSixCharacters\]
     - A file with the same name as the deleted file except starting with $I is added containing metadata about the original file e.g. original directory
       - An I$ file is added each time the file is deleted
       - Interesting from a forensics standpoint even if the R$ file has been deleted
4. When a file is deleted in Apple systems
   - The catalog record (MacOS' Inode equivalent) in the Catalog File is removed (true for the Hierarchical File System and Apple File System (AFS))
   - AFS complicates recovery
     - TRIM -> asynchronous deletion on the SSD (look it up)
     - Native encryption (crypto-erase), if the key is gone -> data (practically) not recoverable
     - Snapshots -> (look it up)
     - Copy-on-write -> Data is not modified in place, instead in an Object Map (look that up)
6. For all of these file systems, the data still sits on it's block even if there is no pointer to it
   - It can, however, be overwritten
   - Crypto-erase -> data not recoverable
7. Bonus:
   - StegFS
   - exFAT (read the paper because this is fun)
---
# Safe disk analysis
## Assuming you receive an img file
### Ideally copy and hash: <br>
1. Makes it easier to ensure you are not working on a faulty copy
   - Flag might have been deleted if improperly accessed
     - Example: mounting the img file
2. Remember: CTF is low stakes, you can always download the file again
3. How to:
  - `md5sum original.img > original.md5`
  - `dd if=/dev/original.img of=/dev/workingcopy.img` <br>
  - `md5sum workingcopy.img > workingcopy.md5` <br>
  - `diff original.md5 workingcopy.md5` -> should give an empty result if identical
<br>
OR: <br>
  - `dc3dd if=/dev/original.img of=/dev/workingcopy.img hash=md5 log=acquire.log` for copying
    - Copying and hashing in one line
    - Error logging file
    - Needs to be installed

---
# Safe disk analysis
## Extracting data
One rule you must always follow: DON'T MOUNT THE DISK IMAGE <br>
It gives problems you would rather not have, use autopsy or the other tools e.g. tsk_recover

---
# Safe disk analysis
## Extracting data
NOTE: I will showcase a lot of information now, but it will make sense in the walkthrough
Commandline tools: <br>
1. tsk_recover (sleuthkit)
  - export files from an image into a local directory
  - easiest way to recover files in the terminal
2. mmls (sleuthkit)
  - gives partition layout of a volume system
  - e.g. start and end bytes of each partition
  - important to set the offset for all other command line tools
3. fls (sleuthkit)
  - list file and directory names in a disk image
4. fsstat (sleuthkit)
  - displays general file system details
5. icat (sleuthkit)
  - output contents of a file based on inode number
6. binwalk (the wild one)
  - `binwalk -e disk.img`
  - Will work for most CTF challenges
  - NOT the forensic standard
  - Carves everything in the file's byte stream regardless of filesystem
    - Caveat: does not care about file system context (deleted data, slack space)
      - Example: missing magic numbers may obfuscate data
      - What is a magic number? - the reason why you should petition for me to make another forensics workshop (i.e. we will ignore it for now)
Another tool: Autopsy
  - Builds on all the sleuthkit tools and gives an ok UI (a little old looking)
  - Does most of the work for you compared to sleuthkit

---
# Safe disk analysis
## How it works in a more realistic forensics case (for those who are interested)
### Scenario: you get a USB (sdb)
1. Ensure auto-mount is disabled
2. Connect the sdb
3. Set the device to read-only: `blockdev --setro /dev/sdb`
4. Create the master img: `dc3dd if=/dev/sdb of=master.img hash=sha256 log=acquire.log`
5. Create a working copy: `dc3dd if=master.img of=workingcopy.img hash=sha256 log=acquire.log`
6. Only work on the working copy!

---
# Challenge walkthrough 1


---
# Slack space
Remember: 
  - A file is allocated some space, when deleted that space can be overwritten
  - A file may have more space allocated than needed e.g. a file may be 120 bytes but a block may be 4096 bytes
Slack space for dummies:
  - Imagine deleting evilThings.txt with 4000 bytes, allocated 1 block on inode 5000
  - The space is now free and can be overwritten
  - Then cuteSafeThings.txt with 120 bytes is allocated to inode 4000
  - byte 121 to 4000 in inode 5000 is content from evilthings.txt!
How do we access it:
  - smart tools built by people smarter than us
  - e.g. Autopsy

---
# Challenge walkthrough 2

---
# Anti-forensics
## I.e. how to cover your tracks
### Journaling
1. Your system logs things you do/your system does
2. Depending on settings it might be cleared on reboot (default is persistent storage)
3. Examples:
   - history: overview of commandline prompts
   - journalctl: overview of processes
4. Goal of an attacker is to manipulate these logs
### Making data unsalvageable
1. shred - (linux) - think of a paper shredder, goes over an changes the file multiple times so it is difficult to find the original content
2. encryption - impossible (if choosing secure crypto) to retrieve without the key, remember to safely handle/delete the key!
### File manipulation
1. Steganography, goal -> hide data in seemingly normal files
### And more: i.e. if you were to think like a malware designer
1. Think about how the processes operate in a system
2. File names and indicators automated detection systems might tag

---
# Sneak peak of a Hack for Snack challenge

---
# Thank you for coming to my ted talk
Questions? <br>
You can also just come up to me and ask me about forensics challenges, software design and/or birds

---
# Resources
[1] Arshad, Humaira & Jantan, Aman & Abiodun, Oludare. (2018). Digital Forensics: Review of Issues in Scientific Validation of Digital Evidence. Journal of Information Processing Systems. 14. 346 ~ 376. 10.3745/JIPS.03.0095. <br>
[2] https://learn.cylabacademy.org/dashboard <br>
[3] https://www.freecodecamp.org/news/file-systems-architecture-explained <br>
