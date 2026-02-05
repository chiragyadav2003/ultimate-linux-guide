# User Management in Linux

## Introduction to User Management in Linux

Linux is a multi-user operating system, meaning multiple users can operate on a system simultaneously. Proper user management ensures security, controlled access, and system integrity.

Key files involved in user management:

- `/etc/passwd` – Stores user account details.
- `/etc/shadow` – Stores encrypted user passwords.
- `/etc/group` – Stores group information.
- `/etc/gshadow` – Stores secure group details.

## Creating Users in Linux

To create a new user in Linux, use:

### `useradd` Command (For most Linux distributions)

```bash
useradd username
```

This creates a user without a home directory.

To create a user with a home directory:

```bash
useradd -m username
```

To specify a shell:

```bash
useradd -s /bin/bash username
```

### `adduser` Command (For Debian-based systems)

```bash
adduser username
```

This is an interactive command that asks for a password and additional details.

## Managing User Passwords

To set or change a user’s password:

```bash
passwd username
```

### Enforcing Password Policies

- **Password expiration**: Set password expiry days

  ```bash
  chage -M 90 username
  ```

- **Lock a user account**

  ```bash
  passwd -l username
  ```

- **Unlock a user account**

  ```bash
  passwd -u username
  ```

## Modifying Users

Modify an existing user with `usermod`:

- Change the username:

  ```bash
  usermod -l new_username old_username
  ```

- Change the home directory:

  ```bash
  usermod -d /new/home/directory -m username
  ```

- Change the default shell:

  ```bash
  usermod -s /bin/zsh username
  ```

## Deleting Users

To remove a user but keep their home directory:

```bash
userdel username
```

To remove a user and their home directory:

```bash
userdel -r username
```

## Working with Groups

### Creating Groups

```bash
groupadd groupname
```

### Adding Users to Groups

```bash
usermod -aG groupname username
```

### Viewing Group Memberships

```bash
groups username
```

### Changing Primary Group

```bash
usermod -g new_primary_group username
```

## Sudo Access and Privilege Escalation

### Adding a User to Sudo Group

On Debian-based systems:

```bash
usermod -aG sudo username
```

On RHEL-based systems:

```bash
usermod -aG wheel username
```

### Granting Specific Commands with Sudo

Edit the sudoers file:

```bash
visudo
```

Then add:

```bash
username ALL=(ALL) NOPASSWD: /path/to/command
```

---

## User Management in Linux - Additional Notes

## 📌 Important Distinctions

### `useradd` vs `adduser`

> **🔑 Key Differences:**
>
> - **`useradd`** is a low-level binary utility - perfect for scripts and automation
> - **`adduser`** is a high-level Perl script wrapper - interactive and user-friendly
>
> **When to use what:**
>
> - Use `useradd` in shell scripts for consistency across distributions
> - Use `adduser` for manual user creation on Debian/Ubuntu systems
>
> **Default behavior:**
>
> - `useradd` does NOT create a home directory by default (use `-m` flag for creating home directory)
> - `adduser` automatically creates home directory and copies skeleton files

---

## 🔐 Password Security Insights

### `/etc/passwd` File

> **📝 Important Notes:**
>
> - Despite the name, does NOT store passwords anymore
> - Passwords moved to `/etc/shadow` for security
> - Contains `x` in password field, pointing to `/etc/shadow`
> - World-readable (needed for basic user info lookup)
>
> **Field structure:**
>
> ```
> username:x:UID:GID:comment:home:shell
> ```

### `/etc/shadow` File

> **⚠️ Critical Security Information:**
>
> - Stores **encrypted** (hashed) passwords, NOT encrypted passwords
> - Uses one-way hashing algorithms (SHA-512, SHA-256, MD5)
> - **Cannot be decrypted** - passwords are verified by comparing hashes
> - Only root has read access (permissions: 640 or 600)
>
> **Password hash format:**
>
> ```
> $id$salt$hashed
> ```
>
> - `$6$` = SHA-512
> - `$5$` = SHA-256
> - `$1$` = MD5

---

## 👥 User and Group ID Ranges

> **📊 Standard UID/GID Conventions:**
>
> - **UID 0** = root user (superuser)
> - **UID 1-999** = system users (services, daemons)
> - **UID 1000+** = regular users (Debian/Ubuntu)
> - **UID 500+** = regular users (RHEL/CentOS 6 and older)
>
> **View your UID/GID:**
>
> ```bash
> id username
> ```

---

## 🏠 Home Directory Creation

> **💡 Best Practices:**
>
> - Always use `-m` flag with `useradd` to create home directory
> - Skeleton files copied from `/etc/skel/` directory
> - Default permissions: 755 (Debian) or 700 (RHEL) depending on `UMASK`
>
> **Skeleton directory (`/etc/skel/`):**
>
> - Contains default configuration files (.bashrc, .profile, etc.)
> - Automatically copied to new user's home directory
> - Customize this directory to set defaults for all new users

---

## 🔒 Account Locking Mechanisms

> **🛡️ Different Ways to Lock Accounts:**
>
> **Method 1: Password lock**
>
> ```bash
> passwd -l username    # Locks by adding ! to password hash
> passwd -u username    # Unlocks account
> ```
>
> **Method 2: Expiry date**
>
> ```bash
> usermod -e 1 username        # Sets expiry to 1970-01-01 (locks immediately)
> usermod -e "" username       # Removes expiry date
> ```
>
> **Method 3: Shell change**
>
> ```bash
> usermod -s /sbin/nologin username    # Prevents login but allows services
> usermod -s /bin/false username       # Completely prevents login
> ```
>
> **Check lock status:**
>
> ```bash
> passwd -S username
> ```

---

## 👤 Special User Accounts

> **⚙️ System Accounts to Know:**
>
> - **nobody** - Unprivileged user for running services
> - **daemon** - Background process owner
> - **bin, sys** - System binaries and files
> - **mail, ftp, www-data** - Service-specific users
>
> **Why they exist:**
>
> - Run services with minimal privileges (security principle of least privilege)
> - Isolate processes from each other
> - Prevent unauthorized access if service is compromised

---

## 🔄 Primary vs Secondary Groups

> **📚 Understanding Group Types:**
>
> **Primary Group:**
>
> - Specified in `/etc/passwd` (4th field - GID)
> - Only ONE primary group per user
> - New files created by user belong to primary group
> - Changed with: `usermod -g groupname username`
>
> **Secondary (Supplementary) Groups:**
>
> - Listed in `/etc/group`
> - User can belong to MULTIPLE secondary groups
> - Used for granting additional permissions
> - Added with: `usermod -aG groupname username`
>
> **⚠️ Warning:** Use `-aG` (append), NOT `-G` alone, or you'll remove existing groups!

---

## 🎯 Password Aging Policies

> **⏰ Complete Password Policy Control:**
>
> **Using `chage` command:**
>
> ```bash
> chage -l username              # List current settings
> chage -M 90 username           # Max 90 days password validity
> chage -m 7 username            # Min 7 days between password changes
> chage -W 14 username           # Warn 14 days before expiry
> chage -I 30 username           # Inactive for 30 days = account lock
> chage -E 2025-12-31 username   # Account expires on specific date
> ```
>
> **Force password change on next login:**
>
> ```bash
> chage -d 0 username
> ```
>
> **System-wide defaults:** Edit `/etc/login.defs`
>
> ```
> PASS_MAX_DAYS   90
> PASS_MIN_DAYS   7
> PASS_WARN_AGE   14
> ```

---

## 📂 Important Configuration Files Reference

> **📋 Quick Reference:**
>
> | File | Purpose | Permissions |
> |------|---------|-------------|
> | `/etc/passwd` | User account information | 644 (world-readable) |
> | `/etc/shadow` | Encrypted passwords | 640 or 600 (root only) |
> | `/etc/group` | Group information | 644 (world-readable) |
> | `/etc/gshadow` | Secure group passwords | 640 or 600 (root only) |
> | `/etc/login.defs` | Default password/account policies | 644 |
> | `/etc/default/useradd` | Default useradd settings | 644 |
>
---

## 🛠️ Useful Commands Quick Reference

> **⚡ Common User Management Commands:**
>
> **User Information:**
>
> ```bash
> id username                    # Show UID, GID, groups
> whoami                         # Current user
> who                            # Logged-in users
> last                           # Login history
> lastlog                        # Last login for all users
> finger username                # Detailed user info (if installed)
> ```
>
> **Group Management:**
>
> ```bash
> groups username                # Show user's groups
> groupmems -g groupname -l      # List group members
> gpasswd -a username group      # Add user to group (alternative)
> gpasswd -d username group      # Remove user from group
> groupdel groupname             # Delete group
> ```
>
> **Account Monitoring:**
>
> ```bash
> passwd -S username             # Password status
> chage -l username              # Password aging info
> faillog -u username            # Failed login attempts
> lastb                          # Bad login attempts
> ```

---

## Switch User and Group Commands

## 🔄 Switch User Commands

### `su` - Switch User

> **Basic Usage:**
>
> ```bash
> su username              # Switch to user (non-login shell)
> su - username            # Switch with login shell (recommended)
> su                       # Switch to root
> su -                     # Switch to root with login shell
> ```
>
> **Key Differences:**
>
> - `su username` - Keeps current environment
> - `su - username` - Loads target user's environment (.bashrc, .profile)
>
> **Exit:** Type `exit` or press `Ctrl+D`

### `sudo` - Execute as Another User

> **Common Usage:**
>
> ```bash
> sudo command             # Run as root
> sudo -u username command # Run as specific user
> sudo -i                  # Root login shell
> sudo -s                  # Root shell (keeps environment)
> sudo su -                # Switch to root (alternative)
> ```
>
> **Key Points:**
>
> - Requires user password (not root password)
> - Logs all commands to `/var/log/auth.log` or `/var/log/secure`
> - Timeout: 15 minutes by default

---

## 👥 Group Switching Commands

### `newgrp` - Switch Primary Group

> **Switch to Different Group:**
>
> ```bash
> newgrp groupname         # Switch primary group temporarily
> exit                     # Return to original group
> ```
>
> **Effect:**
>
> - New files will be created with the new group
> - Only works if user is member of that group
> - Creates new shell session

### `sg` - Execute Command with Different Group

> **Run Single Command:**
>
> ```bash
> sg groupname "command"          # Run command as different group
> sg groupname -c "command"       # Alternative syntax
> ```
>
> **Example:**
>
> ```bash
> sg developers "touch project.txt"    # File owned by developers group
> ```

---

## 📊 Check Current User/Group

> **Quick Commands:**
>
> ```bash
> whoami                   # Current username
> id                       # UID, GID, and all groups
> id -gn                   # Current primary group name
> id -Gn                   # All group names
> groups                   # List all groups for current user
> ```

---

## 🔐 sudo vs su Comparison

> **When to Use What:**
>
> | Feature | `sudo` | `su` |
> |---------|--------|------|
> | Password Required | User's password | Target user's password |
> | Security | Better (logged, limited) | Less secure |
> | Access Control | Granular (sudoers) | All or nothing |
> | Best For | Single commands | Full user session |
> | Audit Trail | Yes (logged) | Limited |
>
> **Best Practice:** Use `sudo` for administrative tasks, `su -` only when necessary

---

## 💡 Practical Examples

> **Common Scenarios:**
>
> **Switch to another user account:**
>
> ```bash
> su - john                # Login as john
> ```
>
> **Run command as different user:**
>
> ```bash
> sudo -u postgres psql    # Run PostgreSQL as postgres user
> ```
>
> **Create file with specific group:**
>
> ```bash
> newgrp developers
> touch newfile.txt        # Owned by developers group
> exit
> ```
>
> **Execute command as root:**
>
> ```bash
> sudo systemctl restart nginx
> ```
>
> **Switch to root shell:**
>
> ```bash
> sudo -i                  # Recommended
> # or
> su -                     # Needs root password
> ```

---

## ⚠️ Important Notes

> **Security Considerations:**
>
> - Always use `su -` (with dash) for complete environment switch
> - Prefer `sudo` over `su` for better audit trails
> - Never share root password - use `sudo` instead
> - `newgrp` creates new shell - remember to `exit` when done
> - Check permissions before switching: `ls -l filename`
>
> **Permission Requirements:**
>
> - `sudo`: User must be in `sudo` or `wheel` group
> - `su`: Must know target user's password
> - `newgrp`: Must be member of target group
