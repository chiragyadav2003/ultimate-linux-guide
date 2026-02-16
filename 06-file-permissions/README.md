# File Permissions Management in Linux

## Introduction to File Permissions

Linux file permissions determine who can read, write, or execute files and directories. Each file and directory has three levels of permission:

- **Owner (User)**: The creator of the file.
- **Group**: Users belonging to the assigned group.
- **Others**: All other users on the system.

Permissions are represented as:

- **Read (`r` or `4`)** – View file contents.
- **Write (`w` or `2`)** – Modify file contents.
- **Execute (`x` or `1`)** – Run scripts or programs.

To check file permissions, use:

```bash
ls -l filename
```

Output example:

```bash
-rwxr--r-- 1 user group 1234 Mar 28 10:00 myfile.sh
```

## Changing Permissions with `chmod`

### Using Symbolic Mode

Modify permissions using symbols:

- Add (`+`), remove (`-`), or set (`=`) permissions.

Examples:

```bash
chmod u+x filename  # Add execute for user
chmod g-w filename  # Remove write for group
chmod o=r filename  # Set read-only for others
chmod u=rwx,g=rx,o= filename  # Set full access for user, read/execute for group, and no access for others
```

### Using Numeric (Octal) Mode

Each permission has a value:

- Read (`4`), Write (`2`), Execute (`1`).

Examples:

```bash
chmod 755 filename  # User (rwx), Group (r-x), Others (r-x)
chmod 644 filename  # User (rw-), Group (r--), Others (r--)
chmod 700 filename  # User (rwx), No access for others
```

## Changing Ownership with `chown`

Modify file owner and group:

```bash
chown newuser filename  # Change owner
chown newuser:newgroup filename  # Change owner and group
chown :newgroup filename  # Change only group
```

Recursively change ownership:

```bash
chown -R newuser:newgroup directory/
```

## Changing Group Ownership with `chgrp`

```bash
chgrp newgroup filename  # Change group
chgrp -R newgroup directory/  # Change group recursively
```

## Special Permissions

### SetUID (`s` on user execute bit)

Allows users to run a file with the file owner's permissions.

```bash
chmod u+s filename
```

Example: `/usr/bin/passwd` allows users to change their passwords.

### SetGID (`s` on group execute bit)

Files: Users run the file with the group's permissions.
Directories: Files created inside inherit the group.

```bash
chmod g+s filename  # Set on file
chmod g+s directory/  # Set on directory
```

### Sticky Bit (`t` on others execute bit)

Used on directories to allow only the owner to delete their files.

```bash
chmod +t directory/
```

Example: `/tmp` directory.

## Default Permissions: `umask`

`umask` defines default permissions for new files and directories.
Check current umask:

```bash
umask
```

Set a new umask:

```bash
umask 022  # Default: 755 for directories, 644 for files
```

---

## Linux File Access - Bank & Account Analogy

> **💡 Key Concept: To access a file, you MUST first have access to its parent folder (directory)**
>
> Just like you need to enter a bank before you can access your account inside it!

---

## 🎯 Quick Answer - Directory Access

> **❓ Question: "What permission do I need to ACCESS a folder?"**
>
> **✅ Answer: Execute (`x`) permission!**
>
> **Breakdown:**
>
> - `r` (read) = List what's inside (`ls`)
> - `w` (write) = Create/delete files inside
> - **`x` (execute) = ENTER the folder** (`cd`) ← **This is what you need!**
>
> **Most common combinations:**
>
> - `r-x` (5) = List + Enter (standard access)
> - `rwx` (7) = Full control
> - `--x` (1) = Enter only (hidden directory)

---

## 🏦 The Bank & Account Analogy

### The Classic Interview Question

**Question:** "If you have read permission on a file but not execute permission on its parent directory, can you read the file?"

**Answer:** **NO!** You cannot access the file at all.

---

## 📊 Understanding the Hierarchy

```
┌─────────────────────────────────────┐
│          Bank Building              │  ← Directory (Folder)
│  ┌───────────────────────────────┐  │
│  │     Account/Safe Deposit      │  │  ← File
│  │      (Your Money/Data)        │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### The Analogy Breakdown

| Linux Concept | Bank Analogy | Explanation |
|---------------|--------------|-------------|
| **Directory (Folder)** | Bank Building | The physical building you must enter first |
| **File** | Account/Safe Deposit Box | Your actual money/valuables inside |
| **Directory `x` permission** | Entry Permission | Can you walk into the bank? |
| **File `r` permission** | Read Account Statement | Can you view your balance? |
| **File `w` permission** | Deposit/Withdraw | Can you modify your account? |

---

## 🔐 The Access Rules

> **⚠️ CRITICAL RULE:**
>
> **To access ANY file, you MUST have execute (`x`) permission on ALL parent directories in the path.**

### Why Execute Permission on Directories?

**Execute (`x`) permission on a directory means:**

- ✅ You can "enter" or "pass through" the directory
- ✅ You can access files inside it (if you have file permissions)
- ✅ You can `cd` into the directory
- ❌ Without `x`, you CANNOT access anything inside, even if you have file permissions

---

## 📝 Real-World Examples

### Example 1: Permission Denied Despite File Access

```bash
# Directory structure
/home/user/documents/secret.txt

# Permissions
drwxr----- user staff /home/user/documents/    # Directory: rwx for owner, r-- for group
-rw-rw-rw- user staff /home/user/documents/secret.txt  # File: rw- for everyone
```

**Scenario:** You are in the `staff` group (not the owner)

**Question:** Can you read `secret.txt`?

**Answer:** **NO!**

**Reason:**

```
Directory permissions: drwxr-----
                         ││││││││
                         │││└─┬─┘
                         │││  └─ Others: --- (no access)
                         ││└──── Group: r-- (read only, NO execute!)
                         │└───── Owner: rwx (full access)
                         └────── File type: d (directory)
```

> **❌ Problem:** Group has `r--` on directory (no `x` execute permission)
>
> **Result:** Cannot enter the directory, so cannot access the file inside!

---

### Example 2: The Execute Permission is Key

```bash
# Same structure
/home/user/documents/secret.txt

# Fixed Permissions
drwxr-x--- user staff /home/user/documents/    # Added 'x' for group!
-rw-rw-rw- user staff /home/user/documents/secret.txt
```

**Now can you read `secret.txt`?**

**Answer:** **YES!**

**Reason:**

```
Directory permissions: drwxr-x---
                         │││││││
                         │││└┬┘
                         │││ └─ Others: --- (no access)
                         ││└─── Group: r-x (read + execute!)
                         │└──── Owner: rwx (full access)
                         └───── File type: d
```

> **✅ Solution:** Group now has `r-x` on directory
>
> **Result:** Can enter directory AND read the file (has rw- permission)

---

## 🎯 Complete Access Path Example

### Scenario: Deep Directory Structure

```
/home/user/projects/web/config/database.conf
```

**To access `database.conf`, you need:**

| Path Component | Required Permission | Why? |
|----------------|-------------------|------|
| `/` | `x` (execute) | Enter root directory |
| `/home` | `x` (execute) | Pass through home |
| `/home/user` | `x` (execute) | Pass through user |
| `/home/user/projects` | `x` (execute) | Pass through projects |
| `/home/user/projects/web` | `x` (execute) | Pass through web |
| `/home/user/projects/web/config` | `x` (execute) | Enter config directory |
| `database.conf` | `r` (read) | Actually read the file |

> **🔑 Key Point:** You need execute (`x`) permission on **EVERY** directory in the path, plus the appropriate permission on the file itself.

---

## 💻 Practical Test

### Test 1: File with Permission, Directory Without

```bash
# Create test structure
mkdir -p /tmp/testbank
echo "Secret data" > /tmp/testbank/account.txt

# Set permissions
chmod 755 /tmp/testbank           # rwxr-xr-x (everyone can enter)
chmod 644 /tmp/testbank/account.txt   # rw-r--r-- (everyone can read file)

# Test: Can read the file
cat /tmp/testbank/account.txt
# Output: Secret data ✅
```

**Now remove execute permission from directory:**

```bash
# Remove execute from directory
chmod 644 /tmp/testbank           # rw-r--r-- (NO execute!)

# Try to read file
cat /tmp/testbank/account.txt
# Output: Permission denied ❌
```

**Result:**

```
cat: /tmp/testbank/account.txt: Permission denied
```

> **📌 Even though the file has `r--` (read) permission for everyone, you CANNOT access it because the directory lacks `x` (execute) permission!**

---

### Test 2: Directory Permission, But No File Permission

```bash
# Reset
chmod 755 /tmp/testbank           # rwxr-xr-x (can enter)
chmod 000 /tmp/testbank/account.txt   # --------- (no permissions!)

# Try to read file
cat /tmp/testbank/account.txt
# Output: Permission denied ❌
```

**Result:**

```
cat: /tmp/testbank/account.txt: Permission denied
```

> **📌 You CAN enter the directory (has `x`), but CANNOT read the file (no `r` permission on file)**

---

## 🧩 Permission Matrix

| Directory `x` | File `r` | Can Read File? | Explanation |
|---------------|----------|----------------|-------------|
| ✅ Yes | ✅ Yes | ✅ **YES** | Can enter directory AND read file |
| ✅ Yes | ❌ No | ❌ **NO** | Can enter directory but file denies read |
| ❌ No | ✅ Yes | ❌ **NO** | Cannot enter directory (blocked at door) |
| ❌ No | ❌ No | ❌ **NO** | Blocked at both levels |

---

## 🎓 Interview Question - Complete Answer

### Question

**"You have 777 permission on a file (`rwxrwxrwx`), but the parent directory has 666 permission (`rw-rw-rw-`). Can you execute the file?"**

### Answer

**NO, you cannot execute the file.**

### Detailed Explanation

```
Directory: drw-rw-rw-  (666)
           │││ ││ ││
           │││ ││ └┴─ Others: rw- (read, write, NO execute)
           │││ └┴──── Group: rw- (read, write, NO execute)
           ││└─────── Owner: rw- (read, write, NO execute)
           │└──────── File type: d (directory)
           └───────── Missing 'x' on directory!

File:      -rwxrwxrwx  (777)
           │└─┬─┘└─┬─┘└─┬─┘
           │  │    │    └─ Others: rwx (full permission)
           │  │    └────── Group: rwx (full permission)
           │  └─────────── Owner: rwx (full permission)
           └────────────── File type: - (regular file)
```

**The Problem:**

1. ✅ File has full `rwx` permissions (777)
2. ❌ Directory lacks `x` (execute) permission
3. ❌ Cannot "enter" the directory to access the file
4. ❌ **Result: Permission denied**

> **🏦 Bank Analogy:** You have the key to your safe deposit box (file permission), but the bank building is locked (no directory execute permission), so you can't get inside to use your key!

---

## 🔍 Quick Reference

### What Each Permission Does

| Permission | On File | On Directory |
|------------|---------|--------------|
| **`r` (read)** | View file contents | List directory contents (`ls`) |
| **`w` (write)** | Modify file contents | Create/delete files inside |
| **`x` (execute)** | Run file as program | **Enter/Access directory** (`cd`) |

---

## 📂 Directory Permissions - Detailed Breakdown

> **🚪 To ACCESS a folder, you need `x` (execute) permission!**
>
> **Not** `r` (read) and **not** `w` (write) - you specifically need **`x`** to enter!

### Understanding Each Directory Permission

#### `r` (read) - See What's Inside

```bash
chmod 004 /tmp/folder   # r-- (read only)

ls /tmp/folder          # ✅ Works! Can list contents
cd /tmp/folder          # ❌ Permission denied (no execute)
cat /tmp/folder/file    # ❌ Permission denied (can't enter to access file)
```

> **📌 Read (`r`) allows:** See filenames inside the directory
>
> **❌ Does NOT allow:** Enter the directory or access files inside

---

#### `w` (write) - Modify Contents

```bash
chmod 002 /tmp/folder   # -w- (write only)

touch /tmp/folder/new.txt     # ❌ Permission denied (no execute to enter)
cd /tmp/folder                # ❌ Permission denied (no execute)
ls /tmp/folder                # ❌ Permission denied (no read)
```

> **📌 Write (`w`) allows:** Create/delete files inside (but ONLY if you also have `x`)
>
> **❌ Does NOT allow:** Enter the directory on its own

---

#### `x` (execute) - Enter/Access Directory

```bash
chmod 001 /tmp/folder   # --x (execute only)

cd /tmp/folder                    # ✅ Works! Can enter
ls /tmp/folder                    # ❌ Permission denied (no read)
cat /tmp/folder/known_file.txt    # ✅ Works! (if you know the filename and have file permission)
```

> **📌 Execute (`x`) allows:**
>
> - Enter the directory with `cd`
> - Access files inside (if you know their names)
> - Traverse through the directory in a path
>
> **⚠️ This is the MINIMUM required to access a directory!**

---

### Directory Permission Combinations

| Permissions | Symbolic | Numeric | What You Can Do |
|-------------|----------|---------|-----------------|
| `---` | No access | 0 | Nothing - completely blocked |
| `--x` | Execute only | 1 | Enter dir, access known files (can't list) |
| `r--` | Read only | 4 | List contents (can't enter or access files) ⚠️ USELESS |
| `r-x` | Read + Execute | 5 | **Common:** List AND enter (normal read access) |
| `-wx` | Write + Execute | 3 | Enter, create/delete (can't list) |
| `rw-` | Read + Write | 6 | List and modify list (can't enter) ⚠️ USELESS |
| `rwx` | Full access | 7 | **Complete control:** List, enter, modify |

---

### Practical Examples

#### Example 1: Execute Only (`--x`)

```bash
mkdir /tmp/secret
chmod 001 /tmp/secret        # --x (execute only)
touch /tmp/secret/data.txt

# Another user trying to access:
cd /tmp/secret               # ✅ Can enter
ls /tmp/secret               # ❌ Cannot list (no read)
cat /tmp/secret/data.txt     # ✅ Can read (if they know the filename!)
```

> **Use case:** Hidden directory - users can access files if they know exact names, but can't browse contents

---

#### Example 2: Read Only (`r--`)

```bash
mkdir /tmp/public
chmod 004 /tmp/public        # r-- (read only)
touch /tmp/public/file.txt

# Another user trying to access:
ls /tmp/public               # ✅ Can see file.txt exists
cd /tmp/public               # ❌ Cannot enter
cat /tmp/public/file.txt     # ❌ Cannot access (blocked at directory)
```

> **Use case:** Almost useless - can see what's inside but can't access it!

---

#### Example 3: Read + Execute (`r-x`) - Most Common

```bash
mkdir /tmp/shared
chmod 005 /tmp/shared        # r-x (read + execute)
touch /tmp/shared/file.txt
chmod 644 /tmp/shared/file.txt

# Another user trying to access:
ls /tmp/shared               # ✅ Can list contents
cd /tmp/shared               # ✅ Can enter
cat /tmp/shared/file.txt     # ✅ Can read file
```

> **Use case:** Standard shared directory - users can browse and access files

---

#### Example 4: Write + Execute (`-wx`)

```bash
mkdir /tmp/dropbox
chmod 003 /tmp/dropbox       # -wx (write + execute)

# Another user can:
cd /tmp/dropbox              # ✅ Can enter
touch /tmp/dropbox/upload.txt # ✅ Can create files
ls /tmp/dropbox              # ❌ Cannot list contents
```

> **Use case:** Drop box directory - users can upload but can't see what others uploaded

---

### Common Directory Permission Patterns

```bash
# Private directory (owner only)
chmod 700 /home/user/private    # rwx------ (7)

# Shared read-only directory
chmod 755 /var/www/html         # rwxr-xr-x (5 for group/others)

# Group collaboration directory
chmod 770 /project/team         # rwxrwx--- (7 for owner/group)

# Drop box (submit only)
chmod 733 /tmp/dropbox          # rwx-wx-wx (3 for group/others)

# Public read directory
chmod 755 /usr/share/doc        # rwxr-xr-x (standard)
```

---

### The Critical Rule

> **⚠️ REMEMBER:**
>
> **To ACCESS any directory, you MUST have `x` (execute) permission!**
>
> - `r` alone = Can see filenames (but can't access)
> - `w` alone = Useless (can't enter to create/delete)
> - `x` alone = Can access files (if you know names)
> - **`r-x` = Standard access** (list + enter)
> - **`rwx` = Full control** (list + enter + modify)

### Critical Directory Permissions

```
rwx = 7  →  Full access (read, write, enter)
r-x = 5  →  Read and enter (common for shared dirs)
--x = 1  →  Enter only (can access files if you know the name)
r-- = 4  →  List only (cannot enter or access files) ⚠️ USELESS for file access
rw- = 6  →  Read and write dir (cannot enter) ⚠️ USELESS for file access
```

---

## ⚡ Key Takeaways

> 1. **Directory `x` permission is MANDATORY** to access any file inside
> 2. **File permissions mean nothing** if you can't reach the file
> 3. **Think hierarchically:** Root → ... → Parent Dir → File
> 4. **Every directory in the path** needs `x` permission
> 5. **Bank analogy:** Directory = Bank Building, File = Account/Safe

---

## 🎯 Common Mistake

```bash
# WRONG THINKING:
"My file has 777, so everyone can access it!"

# CORRECT THINKING:
"My file has 777, but can everyone REACH it through all parent directories?"
```

**Always check the ENTIRE path, not just the file!**

## Conclusion

Understanding file permissions is essential for system security and proper file management. Using `chmod`, `chown`, and `chgrp`, you can control access to files and directories efficiently.

---

## Linux Group Management and File Management - Complete Guide

## Command Symbols & Options Reference

### Common Command Prefixes

- `sudo` → **S**uper**u**ser **do** - Execute command with root privileges

### User/Group Command Options

- `-m` → Create home directory (with useradd)
- `-g` → Set **primary** group (lowercase g)
- `-G` → Set **secondary** groups (uppercase G) - REPLACES all
- `-aG` → **Append** to secondary groups (keeps existing)
- `-a` → Append mode (must be used WITH -G)
- `-d` → Delete/remove (with gpasswd)

### Key Difference

- **Lowercase `-g`** → Primary group (only ONE)
- **Uppercase `-G`** → Secondary groups (multiple allowed)

---

## 1. Creating Users and Groups

### Create Groups

```bash
sudo groupadd developer
```

**Breakdown:**

- `sudo` → Execute with superuser privileges
- `groupadd` → Command to create a new group
- `developer` → Name of the group to create

```bash
sudo groupadd qualityengineer
sudo groupadd aGroup
sudo groupadd bGroup
sudo groupadd cGroup
```

### Create Users with Their Primary Groups

```bash
sudo useradd -m -g aGroup a
```

**Breakdown:**

- `sudo` → Run as superuser (required for user management)
- `useradd` → Command to create a new user
- `-m` → Create home directory (/home/a)
- `-g aGroup` → Set primary group to 'aGroup'
  - `-g` → Primary group option
  - `aGroup` → Group name
- `a` → Username to create

**What it does:** Creates user 'a' with home directory and sets 'aGroup' as primary group

```bash
sudo useradd -m -g bGroup b
sudo useradd -m -g cGroup c
sudo useradd -m -g developer dev
sudo useradd -m -g qualityengineer qe
```

**Verify:**

```bash
id a
```

**Output:**

```
uid=1001(a) gid=1001(aGroup) groups=1001(aGroup)
```

---

## 2. Understanding `-G` vs `-aG`

### ⚠️ `-G` (DANGEROUS - Replaces all secondary groups)

```bash
sudo usermod -G developer dev
```

**Breakdown:**

- `sudo` → Run as superuser
- `usermod` → Modify existing user
- `-G developer` → Set secondary groups (REPLACES all existing)
  - `-G` → Secondary groups option (overwrites)
  - `developer` → New group list
- `dev` → Username to modify

**Result:** Removes ALL existing secondary groups, sets only `developer`

---

### ✅ `-aG` (SAFE - Appends to existing groups)

```bash
sudo usermod -aG developer dev
```

**Breakdown:**

- `sudo` → Run as superuser
- `usermod` → Modify existing user
- `-aG developer` → Append to secondary groups (keeps existing)
  - `-a` → Append mode (use WITH -G)
  - `-G` → Secondary groups option
  - `developer` → Group to add
- `dev` → Username to modify

**Result:** ADDS `developer` to existing groups without removing others

**Always use `-aG` unless you want to remove all other groups!**

---

## 3. Your Specific Requirements

### For `dev` user: Only in `developer` group

**Option 1: Change primary group**

```bash
sudo usermod -g developer dev
```

**Verify:**

```bash
id dev
```

**Output:**

```
uid=1004(dev) gid=1004(developer) groups=1004(developer)
```

**Option 2: Keep primary group + add secondary**

```bash
sudo usermod -aG developer dev
```

**Output:**

```
uid=1004(dev) gid=1006(dev) groups=1006(dev),1004(developer)
```

---

### For `qe` user: In both `qe` (primary) and `qualityengineer` groups

**Add qualityengineer as secondary group:**

```bash
sudo usermod -aG qualityengineer qe
```

**Verify:**

```bash
id qe
```

**Output:**

```
uid=1005(qe) gid=1007(qe) groups=1007(qe),1005(qualityengineer)
```

---

## 4. Changing Primary Group

### Change Primary Group

```bash
sudo usermod -g newGroupName username
```

**Breakdown:**

- `sudo` → Run as superuser
- `usermod` → Modify existing user
- `-g newGroupName` → Change primary group
  - `-g` → Primary group option (lowercase 'g')
  - `newGroupName` → New primary group name
- `username` → User to modify

**What it does:** Changes the user's primary group (affects new file ownership)

**Example:**

```bash
sudo usermod -g developer a
```

**Verify:**

```bash
id a
```

**Before:**

```
uid=1001(a) gid=1001(aGroup) groups=1001(aGroup)
```

**After:**

```
uid=1001(a) gid=1004(developer) groups=1004(developer)
```

---

## 5. Keep User in ONLY One Group (Remove All Others)

### Method 1: Change primary group (keeps secondary groups)

```bash
sudo usermod -g targetGroup username
```

**Breakdown:**

- Changes **only** the primary group

- Does **NOT** remove secondary groups
- Old primary group is replaced, secondary groups remain

**Example:**

```bash
# Before: gid=1004(bGroup) groups=1004(bGroup),1001(developer),1003(aGroup)
sudo usermod -g qualityengineer b
# After: gid=1002(qualityengineer) groups=1002(qualityengineer),1001(developer),1003(aGroup)
# Notice: developer and aGroup (secondary) still exist!
```

### Method 2: Remove ALL secondary groups

```bash
sudo usermod -G "" username
```

**Breakdown:**

- `sudo` → Run as superuser
- `usermod` → Modify user
- `-G ""` → Set secondary groups to empty (removes all)
  - `-G` → Secondary groups option
  - `""` → Empty string (no groups)
- `username` → User to modify

**What it does:** Removes ALL secondary groups, keeps only primary group

### Method 3: Remove from specific secondary group

```bash
sudo gpasswd -d username groupname
```

**Breakdown:**

- `sudo` → Run as superuser
- `gpasswd` → Group password administration tool
- `-d username` → Delete user from group
  - `-d` → Delete/remove option
  - `username` → User to remove
- `groupname` → Group to remove user from

**What it does:** Removes user from ONE specified secondary group (doesn't affect primary or other secondary groups)

**Example - Keep `a` only in `aGroup`:**

```bash
# Step 1: Set aGroup as primary
sudo usermod -g aGroup a

# Step 2: Remove ALL secondary groups
sudo usermod -G "" a
```

**Verify:**

```bash
id a
```

**Output:**

```
uid=1001(a) gid=1001(aGroup) groups=1001(aGroup)
```

---

## 6. Group Management Commands

### Create Group

```bash
sudo groupadd groupname
```

### Delete Group

```bash
sudo groupdel groupname
```

**Note:** Cannot delete a group that is the primary group of any user

### Rename Group

```bash
sudo groupmod -n newname oldname
```

### View Group Members

```bash
getent group groupname
```

**Output:**

```
developer:x:1004:dev,john,sarah
```

### List All Groups

```bash
cat /etc/group
```

---

## 7. Complete Examples

### Example 1: Setup users a, b, c with their groups

```bash
# Create groups
sudo groupadd aGroup
sudo groupadd bGroup
sudo groupadd cGroup

# Create users with primary groups
sudo useradd -m -g aGroup a
sudo useradd -m -g bGroup b
sudo useradd -m -g cGroup c

# Set passwords
sudo passwd a
sudo passwd b
sudo passwd c
```

**Verify all users:**

```bash
id a && id b && id c
```

**Output:**

```
uid=1001(a) gid=1001(aGroup) groups=1001(aGroup)
uid=1002(b) gid=1002(bGroup) groups=1002(bGroup)
uid=1003(c) gid=1003(cGroup) groups=1003(cGroup)
```

---

### Example 2: Add user to multiple secondary groups

```bash
# Add user 'a' to developer and qualityengineer groups
sudo usermod -aG developer,qualityengineer a
```

**Verify:**

```bash
id a
```

**Output:**

```
uid=1001(a) gid=1001(aGroup) groups=1001(aGroup),1004(developer),1005(qualityengineer)
```

---

### Example 3: Remove user from a specific group

```bash
# Remove 'a' from developer group
sudo gpasswd -d a developer
```

**Output:**

```
Removing user a from group developer
```

**Verify:**

```bash
id a
```

**Output:**

```
uid=1001(a) gid=1001(aGroup) groups=1001(aGroup),1005(qualityengineer)
```

---

## 8. Quick Reference Table

| Task | Command |
|------|---------|
| Create group | `sudo groupadd groupname` |
| Delete group | `sudo groupdel groupname` |
| Add user to group (append) | `sudo usermod -aG groupname username` |
| Replace all secondary groups | `sudo usermod -G group1,group2 username` |
| Change primary group | `sudo usermod -g groupname username` |
| Remove from specific group | `sudo gpasswd -d username groupname` |
| View user's groups | `id username` or `groups username` |
| View group members | `getent group groupname` |
| Remove all secondary groups | `sudo usermod -G "" username` |

---

## 9. Key Differences Summary

### `-G` vs `-aG`

```bash
# Scenario: User 'a' is in groups: aGroup (primary), developer, tester

# Using -G (REPLACES all secondary groups)
sudo usermod -G qualityengineer a
# Result: aGroup (primary), qualityengineer
# Lost: developer, tester

# Using -aG (APPENDS to existing groups)
sudo usermod -aG qualityengineer a
# Result: aGroup (primary), developer, tester, qualityengineer
# Lost: nothing
```

**Rule of thumb:** Always use `-aG` unless you explicitly want to remove groups

---

### `-g` (Primary Group Change) - Real Example

```bash
# Initial state
id b
# uid=1002(b) gid=1004(bGroup) groups=1004(bGroup),1001(developer),1003(aGroup)

# Change primary group to qualityengineer
sudo usermod -g qualityengineer b

# After
id b
# uid=1002(b) gid=1002(qualityengineer) groups=1002(qualityengineer),1001(developer),1003(aGroup)
```

**Key observation:**

- ✅ Primary changed: `bGroup` → `qualityengineer`
- ✅ Secondary groups **NOT removed**: `developer`, `aGroup` still present
- ⚠️ Old primary `bGroup` disappeared (it was ONLY primary, not secondary)

---

## 10. File Ownership and Permissions

### Understanding File Ownership

When you create a file, it gets TWO owners:

1. **User owner** → The user who created it
2. **Group owner** → The user's **current primary group**

```bash
# User 'a' with primary group 'aGroup'
su - a
touch file1.txt
ls -l file1.txt
```

**Output Breakdown:**

```
-rw-r--r--  1  a  aGroup  0  Feb 06 10:00  file1.txt
│           │  │  │       │  │             │
│           │  │  │       │  │             └─ Filename
│           │  │  │       │  └─ Timestamp
│           │  │  │       └─ File size (bytes)
│           │  │  └─ Group owner (primary group of creator)
│           │  └─ User owner (who created the file)
│           └─ Number of hard links
└─ File permissions (10 characters)
```

**File Permissions Breakdown:**

```
-rw-r--r--
│ │  │  │
│ │  │  └─ Others: r-- (read only)
│ │  └──── Group:  r-- (read only)
│ └─────── Owner:  rw- (read + write)
└───────── File type: - (regular file)

File type symbols:
-  = Regular file
d  = Directory
l  = Symbolic link
b  = Block device
c  = Character device
p  = Named pipe
s  = Socket
```

**Permission breakdown by position:**

```
Position:  1  2 3 4  5 6 7  8 9 10
Symbol:    -  r w -  r - -  r - -
           │  └─┬─┘  └─┬─┘  └─┬─┘
           │    │      │      └─ Others (everyone else)
           │    │      └─ Group (members of file's group)
           │    └─ Owner (user who owns the file)
           └─ File type

Each permission set has 3 characters:
r (read)    = 4
w (write)   = 2
x (execute) = 1
- (none)    = 0
```

---

### Primary Group Determines New File Ownership

**Current primary group = Group owner of new files**

```bash
# Check user 'a' details
id a
# uid=1001(a) gid=1001(aGroup) groups=1001(aGroup),1004(developer)
#             ^^^^^^^^^^^^^^^^
#             This is the primary group that will own new files

# Create file
touch file1.txt

# Check ownership
ls -l file1.txt
# -rw-r--r-- 1 a aGroup 0 Feb 06 10:00 file1.txt
#              └─┬─────┘
#                └─ Group owner = primary group (aGroup)
```

---

### Temporarily Change Primary Group for File Creation

**Use `newgrp` to switch primary group temporarily:**

```bash
# Before: primary group is aGroup
id -gn
# Output: aGroup

# Switch primary group to developer
newgrp developer

# Now primary group is developer
id -gn
# Output: developer

# Create file - will be owned by developer group
touch file2.txt
ls -l file2.txt
# -rw-r--r-- 1 a developer 0 Feb 06 10:01 file2.txt
#              └─┬────────┘
#                └─ Group owner changed to developer

# Exit newgrp session (returns to original primary group)
exit

# Back to aGroup
id -gn
# Output: aGroup
```

**How `newgrp` works:**

- Creates a **new shell** with different primary group
- Only works if you're a **member** of that group
- Changes are **temporary** (only in that shell session)
- Type `exit` to return to original primary group

---

### Permanent vs Temporary Primary Group Change

| Method | Command | Duration | Scope |
|--------|---------|----------|-------|
| **Permanent** | `sudo usermod -g newGroup user` | Until changed again | All future logins |
| **Temporary** | `newgrp newGroup` | Current shell only | Until `exit` |

**Example - Permanent Change:**

```bash
# Permanently change primary group
sudo usermod -g developer a

# User must re-login for system-wide effect
# All new files will now be owned by developer group
```

**Example - Temporary Change:**

```bash
# Temporarily switch to developer group
newgrp developer

# Create file
touch temp.txt
ls -l temp.txt
# -rw-r--r-- 1 a developer ...

# Exit newgrp (back to original primary group)
exit

# Create another file
touch temp2.txt
ls -l temp2.txt
# -rw-r--r-- 1 a aGroup ...
```

---

### File Permission Numbers (chmod)

**Permission values:**

- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1

**Three digit format: Owner Group Others**

```
chmod 740 file.txt
      │││
      ││└─ Others: 0 = --- (no permissions)
      │└── Group:  4 = r-- (read only)
      └─── Owner:  7 = rwx (read, write, execute)
```

**Common permission combinations:**

```
7 = rwx = 4+2+1 = Read, Write, Execute
6 = rw- = 4+2   = Read, Write
5 = r-x = 4+1   = Read, Execute
4 = r-- = 4     = Read only
0 = --- = 0     = No permissions
```

---

### Who Gets Which Permissions?

When accessing a file, Linux checks in this order:

**1. Are you the owner?**

```bash
ls -l file.txt
# -rwxr--r-- 1 alice developers ...
#  ^^^
#  Use OWNER permissions (rwx)
```

If you're `alice`, you get **rwx** regardless of group membership

**2. Are you in the group?**

```bash
ls -l file.txt
# -rwxr--r-- 1 alice developers ...
#     ^^^
#     Use GROUP permissions (r--)
```

If you're NOT alice but in `developers` group, you get **r--**

**3. Everyone else**

```bash
ls -l file.txt
# -rwxr--r-- 1 alice developers ...
#        ^^^
#        Use OTHERS permissions (r--)
```

If you're NOT alice and NOT in `developers`, you get **r--**

---

### Practical Example - Complete Workflow

```bash
# Scenario: User 'dev' in groups: dev(primary), developer, tester

# Check current state
id dev
# uid=1004(dev) gid=1006(dev) groups=1006(dev),1004(developer),1005(tester)

# Create file with default primary group
touch project1.txt
ls -l project1.txt
# -rw-r--r-- 1 dev dev ...
#                  ^^^
#                  Owned by 'dev' group (primary)

# Switch to developer group temporarily
newgrp developer
id -gn
# developer

# Create file with developer group
touch project2.txt
ls -l project2.txt
# -rw-r--r-- 1 dev developer ...
#                  ^^^^^^^^^
#                  Owned by 'developer' group

# Set specific permissions
chmod 750 project2.txt
ls -l project2.txt
# -rwxr-x--- 1 dev developer ...
#  ^^^^^^^^
#  Owner:  rwx (7) - full access
#  Group:  r-x (5) - read, execute
#  Others: --- (0) - no access

# Exit newgrp
exit

# Create another file (back to dev group)
touch project3.txt
ls -l project3.txt
# -rw-r--r-- 1 dev dev ...
```

---

### Changing File Ownership

**Change user owner:**

```bash
sudo chown newuser file.txt
```

**Change group owner:**

```bash
sudo chown :newgroup file.txt
# OR
sudo chgrp newgroup file.txt
```

**Change both:**

```bash
sudo chown newuser:newgroup file.txt
```

**Example:**

```bash
ls -l file.txt
# -rw-r--r-- 1 alice developers ...

# Change to bob:testers
sudo chown bob:testers file.txt

ls -l file.txt
# -rw-r--r-- 1 bob testers ...
```

---

## ⚠️ Important Notes

1. **Group changes take effect on next login** - existing sessions keep old groups
2. **Cannot delete a group** if it's the primary group of any user
3. **Use `-aG` not `-G`** to avoid accidentally removing groups
4. **Primary group** determines ownership of newly created files
5. **User must re-login** to see group membership changes in active sessions
