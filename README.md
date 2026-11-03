# Digital Forensics – Exam Answers (Unit 4 & Unit 5)

## Table of Contents
1. [Live Acquisition: Computer ON with Encrypted Drives](#q1-live-acquisition-computer-on-with-encrypted-drives)
2. [RAID Levels and Forensic Challenges](#q2-raid-levels-and-forensic-challenges)
3. [Data-Hiding Techniques and Detection](#q3-data-hiding-techniques-and-detection)
4. [AFF vs RAW Image Formats](#q4-aff-vs-raw-image-formats)
5. [Password-Cracking Techniques](#q5-password-cracking-techniques)
6. [ProDiscover Acquisition Workflow](#q6-prodiscover-acquisition-workflow)
7. [Validating Forensic Data](#q7-validating-forensic-data)
8. [Host Protected Area (HPA)](#q8-host-protected-area-hpa)

---

## Q1. Live Acquisition: Computer ON with Encrypted Drives

**Question:** A forensic examiner finds a computer powered ON with encrypted drives at a cyber-fraud scene. What acquisition strategy should be applied, and why? Compare the risks of powering off.

### Recommended Strategy: Live Acquisition
While the system is ON, the encrypted drives are **unlocked** and the **encryption keys are in RAM**. RAM is lost when power goes off, so the examiner must collect data **before shutting down**.

### Steps
1. **Document the scene:** photograph the screen, note open programs, date and time; start chain of custody.
2. **Avoid unnecessary actions:** every click changes data.
3. **Capture RAM first** using FTK Imager, winpmem, or Volatility to recover encryption keys (BitLocker, LUKS).
4. **Image the unlocked drives** (full or logical acquisition).
5. **Hash everything** (MD5/SHA-256).
6. **Record every action**, since live acquisition changes some metadata.
7. Only then decide whether to shut down.

### Risks of Powering Off

| Risk | Explanation |
|---|---|
| Drives get locked | Data unreadable without password/key |
| Keys lost | Keys in RAM disappear |
| Volatile evidence lost | Processes, network connections, sessions gone |
| Cracking may be impossible | AES-256 can take years to brute force |
| Case may fail | Key fraud evidence never recovered |

### Comparison

| | Live Acquisition | Power OFF (Static) |
|---|---|---|
| Encrypted data | Accessible | Locked |
| RAM / keys | Captured | Lost |
| Data changes | Small, must be documented | None |
| Repeatable | Less | Fully |

**Conclusion:** Live acquisition is correct here. Small metadata changes are acceptable compared to losing all encrypted evidence.

---

## Q2. RAID Levels and Forensic Challenges

**Question:** Explain the key RAID levels and evaluate the forensic challenges of each.

**RAID** (Redundant Array of Independent Disks) combines several physical disks into **one logical disk** for speed, capacity, or safety.

| RAID Level | How it works | Forensic challenges |
|---|---|---|
| **RAID 0** (Striping) | Data split across disks, no backup | Need **all disks** plus stripe order, size, and offset; one disk failure means data loss |
| **RAID 1** (Mirroring) | Same data on two disks | **Easiest**: one disk has everything |
| **RAID 3/4** (Dedicated parity) | Striping plus one parity disk | Parity disk **must be included**; controller details matter |
| **RAID 5** (Distributed parity) | Data and parity spread over all disks | Survives 1 failure; need stripe size, order, **parity rotation** |
| **RAID 6** (Double parity) | Like RAID 5 with two parity blocks | Survives 2 failures; more complex rebuild |
| **Nested** (RAID 10, 15) | Mix of mirroring and striping/parity | **Most complex**: all settings needed |

### General Considerations
1. Identify RAID type (hardware or software).
2. Capture **all member disks** with write-blockers.
3. Document disk order, slots, serial numbers, controller settings; take photos.
4. Plan for very large storage needs.
5. Check tool support: ProDiscover images each disk separately and creates a **.pdg group file** for virtual rebuild.
6. Recovery tools (Runtime RAID Reconstructor, R-Studio) may modify data, so use them **only on copies**.
7. If too large, use **logical or sparse acquisition** and document why.
8. Hash every image and record every rebuild step.

**Conclusion:** The more complex the RAID, the harder the reconstruction. Capturing all disks and documenting everything is essential.

---

## Q3. Data-Hiding Techniques and Detection

**Question:** Outline the primary data-hiding techniques used in cybercrime and how forensic software identifies hidden evidence.

**Data hiding** means changing files or storage structures to conceal information from investigators.

### Techniques
1. **Changing file extensions:** e.g., `.xlsx` renamed to `.jpg`.
2. **Hidden attributes:** files marked "Hidden" in Windows.
3. **Hiding partitions:** drive letter removed using `diskpart`.
4. **Bad-cluster marking:** good clusters marked "bad" (Norton DiskEdit) so the OS ignores them; persists after NTFS conversion.
5. **Bit-shifting:** bits shifted left/right to scramble data.
6. **Encryption / password protection:** AES, PGP, locked archives.
7. **Steganography:** data hidden inside images, audio, or video.

### Detection

| Technique | Detection method |
|---|---|
| Changed extensions | Check **file header/signature** vs extension |
| Hidden attributes | Forensic tools list all files |
| Hidden partitions | Check partition table gaps / unallocated space; mount hidden partition |
| Bad clusters | **WinHex, X-Ways, ProDiscover, OSForensics** read clusters directly |
| Bit-shifting | Shift bits back in hex editors; confirm with **MD5 hash** |
| Encryption | RAM capture; Passware, PRTK, Hashcat, John the Ripper |
| Steganography | Steganalysis: histogram, LSB checks, compare with original |

### Investigator Safeguards
- Make sure **total disk space adds up**.
- Check slack and unallocated space.
- Verify partition tables.
- Use **NIST NSRL / KFF** hash sets to filter known files.

**Conclusion:** Forensic tools uncover hidden data by checking signatures, reading raw disk areas, comparing hashes, and using specialised analysis.

---

## Q4. AFF vs RAW Image Formats

**Question:** A drive contains an AFF (.afd & .afm) image and a RAW (.dd) image. Compare usage scenarios and when each is more suitable.

### AFF (Advanced Forensic Format)
- Open-source, created by **Dr. Simson Garfinkel**.
- **.afd** holds image data; **.afm** holds metadata.
- Compression, embedded metadata, **no size limit**, **self-checking**, cross-platform.

### RAW (.dd)
- Simple bit-by-bit copy (Linux `dd`).
- No compression, no metadata; hash stored separately.

| Feature | AFF | RAW |
|---|---|---|
| Compression | Yes | No |
| Storage | Less | Same as original disk |
| Metadata | Built-in | None |
| Integrity check | Built-in | Manual hash |
| Tool support | Not all commercial tools | Almost all tools |
| Speed | Slightly slower | Fast |

### When to Use
**AFF:** limited storage, very large images, metadata must stay with the image, open-source or cross-platform work, long-term archiving.

**RAW:** maximum tool compatibility, quick simple imaging, sharing with others who use different tools.

### Practical Steps
1. Verify both images (AFF self-checks; RAW against its hash file).
2. Open AFF in a supporting tool (OSForensics, Autopsy/Sleuth Kit).
3. Open RAW in any forensic tool.
4. If needed, convert AFF to raw and hash again.

**Conclusion:** AFF is best for storage, metadata, and integrity; RAW is best for compatibility and simplicity.

---

## Q5. Password-Cracking Techniques

**Question:** Explain password-cracking techniques including hybrid attacks, hash-based attacks, rainbow tables, and salting.

### 1. Hybrid Attack
- Combines **dictionary and brute force**.
- Build a suspect profile: family/pet names, birthdays, teams, hobbies, schools.
- Add numbers and symbols.
- **Example:** dog named Rocky gives `Rocky123`, `Rocky#@`, `Rocky2024`.
- Works because people use familiar personal info.

### 2. Hash-Based Attack
- Passwords are stored as **hashes** (MD5, SHA-1, SHA-256), which are one-way.
- Hash each guess and compare with the stored hash.
- Slow, because every guess must be hashed.

### 3. Rainbow Tables
- Huge **pre-computed** database of password hashes; attacker just looks up the hash.
- Fast and effective on **unsalted** hashes (e.g., Project RainbowCrack).
- Needs huge storage; **useless against salted hashes**.

### 4. Salting
- Random characters added before hashing.
- **Example:** `apple` + salt `Xy2@` gives `Hash(appleXy2@)`.
- Same password gives different hashes, which **defeats rainbow tables**.

**Common tools:** John the Ripper, Hashcat, Ophcrack, Passware, AccessData PRTK.

**Conclusion:** Hybrid uses smart guessing, hash attacks compare fingerprints, rainbow tables speed things up, and salting protects against rainbow tables.

---

## Q6. ProDiscover Acquisition Workflow

**Question:** Summarize ProDiscover's proprietary acquisition workflow and the primary files it produces.

### Workflow
1. **Start:** Connect drive via write-blocker; run ProDiscover as administrator; **Action → Capture Image**; select source (e.g., PhysicalDrive1).
2. **Destination:** Choose folder and file name (e.g., InChp031).
3. **Split:** 650 MB for CD; 2 GB for FAT32; larger sizes need NTFS.
4. **Format:** ProDiscover **.eve** (with metadata: technician, image number, description; optional compression) or **raw (dd)**.
5. **Image and verify** the hash.

### Output Files

| File | Purpose |
|---|---|
| `.eve` | Main image file |
| `-Split1.eve`, `-Split2.eve`... | Additional segments |
| `.log` | Errors and warnings |
| `.pds` | Inventory: order of segments |
| `.md5` | Hash file (raw format only) |
| `.pdg` | RAID group file for virtual rebuild |

### .eve vs Raw

| | .eve | Raw |
|---|---|---|
| Metadata | Embedded | None |
| Compression | Yes | No |
| Self-check | Built-in | Separate .md5 |
| Compatibility | Mainly ProDiscover | Almost all tools |

**Conclusion:** ProDiscover's workflow produces image, log, inventory, and hash files that keep evidence complete and verifiable.

---

## Q7. Validating Forensic Data

**Question:** Explain the importance of validating forensic data and the techniques used in tools and hex editors.

### Why Validate
1. Proves evidence is **identical** to the original.
2. Detects tampering: **one changed bit changes the hash**.
3. Required for **court admissibility**.
4. Supports **chain of custody**.
5. Builds credibility.

### Hash Algorithms

| Algorithm | Size | Note |
|---|---|---|
| MD5 | 128-bit | Fast, older |
| SHA-1 | 160-bit | Stronger |
| SHA-256 | 256-bit | Recommended |

**Process:** Hash original, create image, hash image, confirm match, recheck after every transfer.

### Validation in Tools

| Tool | Method |
|---|---|
| ProDiscover | Hash stored in .eve; **recalculated on load**; warns on mismatch |
| FTK Imager | Hash embedded in .e01 |
| SMART | Hash embedded in .s01 |
| OSForensics | Generates and re-verifies MD5/SHA-1 |
| Raw (.dd) | **Manual** hash calculation and periodic recheck |

### Hex Editors (WinHex)
- Hash whole drives, files, or sectors.
- **Block-wise hashing** finds fragments of deleted files in slack/unallocated space.
- Proves known contraband exists even if partly deleted.

### Hash Databases
**NIST NSRL** and **AccessData KFF** filter out known safe files and highlight suspicious ones.

**Conclusion:** Hashing, tool verification, and hex-editor techniques make evidence authentic, untampered, and court-ready.

---

## Q8. Host Protected Area (HPA)

**Question:** You discover an HPA not visible in the OS. Analyse why it may contain critical evidence and explain how to acquire it safely.

**HPA** is a hidden, reserved area of a hard disk **not visible to the OS or normal BIOS**. Manufacturers use it for recovery tools, but suspects can hide data there.

### Why It May Hold Critical Evidence
1. Invisible to users and normal tools, so it's an ideal hiding place.
2. Standard imaging **skips it**.
3. Survives formatting and OS reinstall.
4. A deliberately created HPA suggests **intent to hide**.
5. Missing it gives an incomplete image, which weakens the case.

### Detection
Compare the **real sector count** (manufacturer specs) with the size shown in **BIOS/Disk Management**. A difference suggests an HPA.

### Safe Acquisition Steps
1. **Document:** model, serial number, reported vs actual size; photos; chain of custody.
2. **Hardware write-blocker** that supports HPA.
3. **HPA-capable tools:** ProDiscover (with supporting write-blocker), ImageMASSter Solo, X-Ways Replica, or tools that set exact sector count.
4. **Image the full physical disk**, including HPA.
5. **Hash and verify** source and image.
6. **Analyse HPA** with hex editors or forensic suites.
7. **Record** tools, settings, sector counts, and commands.

**Caution:** Removing or changing the HPA modifies the drive. Avoid this, or document it fully if unavoidable.

**Conclusion:** Detecting the HPA by comparing sector counts and capturing it with write-blockers and HPA-capable tools ensures a complete, defensible image.

---

## Quick Revision Table

| Q | Topic | Key Points |
|---|---|---|
| 1 | Live acquisition | Capture RAM first; powering off locks drives and loses keys |
| 2 | RAID | Capture all disks; record stripe size, order, parity |
| 3 | Data hiding | Signatures, raw sector reads, hash comparison, steganalysis |
| 4 | AFF vs RAW | AFF = compressed + metadata; RAW = universal + simple |
| 5 | Password cracking | Hybrid, hash comparison, rainbow tables, salting defeats tables |
| 6 | ProDiscover | .eve, -SplitN.eve, .log, .pds, .md5, .pdg |
| 7 | Validation | Hashes must match; raw needs manual hashing; NSRL/KFF filtering |
| 8 | HPA | Compare sector counts; HPA-capable tools + write-blocker |
