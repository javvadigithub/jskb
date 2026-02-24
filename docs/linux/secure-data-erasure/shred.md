# Shred Utility – Secure Data Erasure in Linux

---

## 1. Concept

`shred` is a GNU Core Utility used to securely delete files by overwriting their data blocks multiple times before removing the file reference.

In traditional file deletion (`rm`):

* Only the inode reference is removed.
* Data blocks remain intact.
* Data is recoverable using forensic tools.

`shred` mitigates **data remanence** by overwriting the physical storage blocks associated with a file.

---

## 2. Architecture / Working Principle

### File Deletion (Logical)

```
Disk
 ├── MBR
 ├── Partition Table
 ├── File System
 │     ├── inode removed
 │     └── data blocks still present
```

Result: Recoverable

---

### Shred Process (Physical Overwrite)

```
Data blocks:
[Original Data]

Pass 1 → Random overwrite
Pass 2 → Random overwrite
Pass 3 → Random overwrite
Final Pass → Zero fill (optional)

inode removed
```

Result: Data blocks destroyed (HDD effective)

---

### Key Technical Understanding

* `shred` operates at **file-system block level**
* It does NOT modify:

  * MBR
  * Partition table
  * Bootloader
* Unless applied to entire disk device

---

## 3. Command Syntax

### Basic Usage

```bash
shred file.txt
```

Default: 3 overwrite passes.

---

### Specify Overwrite Count

```bash
shred -n 5 file.txt
```

`-n` → number of overwrite passes.

---

### Recommended Secure Deletion

```bash
shred -n 3 -z -u confidential.txt
```

Options:

* `-n` → overwrite count
* `-z` → final zero overwrite
* `-u` → remove file after overwrite

---

### Verbose Mode

```bash
shred -v -n 3 file.txt
```

Shows overwrite progress.

---

### Entire Disk Wipe (Destructive)

```bash
sudo shred -v -n 3 /dev/sdb
```

This overwrites:

* MBR
* Partition table
* File system metadata
* All data blocks

System becomes unbootable.

---

## 4. Lab Demonstration

### Step 1 – Create Test File

```bash
echo "Sensitive Data" > secret.txt
```

---

### Step 2 – Verify File Exists

```bash
ls -l secret.txt
```

---

### Step 3 – Run Shred

```bash
shred -n 3 -z -u secret.txt
```

---

### Step 4 – Verify Deletion

```bash
ls secret.txt
```

Expected output:

```
No such file or directory
```

---

## 5. Security Implications

### Red Team Perspective

* Prevents recovery of sensitive artifacts.
* Useful for anti-forensics.
* Can destroy forensic evidence if misused.

---

### Blue Team Perspective

* Helps in secure data disposal.
* Used in system decommissioning.
* Aligns with media sanitization policies.

---

## 6. HDD vs SSD Consideration

### HDD (Magnetic Disk)

* Data stored in fixed sectors.
* Overwrite replaces magnetic pattern.
* `shred` is effective.

---

### SSD (Flash Storage)

* Uses wear leveling.
* Data remapping occurs internally.
* Overwrite may not hit original cells.

Result: `shred` is unreliable on SSD.

Recommended alternatives:

* Full disk encryption
* ATA Secure Erase
* NVMe format command

---

## 7. Relation to MBR

### MBR Structure (First 512 Bytes of Disk)

```
| Boot Code | Partition Table | Signature |
```

If you run:

```bash
shred file.txt
```

MBR remains untouched.

If you run:

```bash
sudo shred /dev/sda
```

Then:

* MBR overwritten
* Partition table destroyed
* Bootloader erased
* System unbootable

---

## 8. Risk & Mitigation

| Risk                          | Impact                   |
| ----------------------------- | ------------------------ |
| Running shred on wrong device | Full data loss           |
| Using on SSD                  | Ineffective sanitization |
| Using on mounted system disk  | System corruption        |

Mitigation:

* Always verify device using `lsblk`
* Use encryption-first strategy
* Follow NIST SP 800-88 sanitization guidelines

---

## 9. Summary

* `shred` performs multi-pass overwrite.
* Effective on HDD.
* Not reliable on SSD.
* Does not affect MBR unless entire disk is targeted.
* Useful for secure deletion and anti-forensics scenarios.

---

