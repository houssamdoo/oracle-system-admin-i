# LAB - 02

## Exercise 1: Creating Users and Understanding UPG (User Private Groups)

### Goal
Create a new user and observe how Oracle Linux 8 implements User Private Groups (UPG) by default.

### Instructions

1. Create a new user named `devuser1`:
   ```bash
   useradd devuser1
   ```

2. Set a password for the user:
   ```bash
   passwd devuser1
   # Enter a temporary password like "TempPass123!"
   ```

3. Inspect the `/etc/passwd` and `/etc/group` entries:
   ```bash
   grep devuser1 /etc/passwd
   grep devuser1 /etc/group
   ```

4. Verify the home directory and default shell:
   ```bash
   ls -ld /home/devuser1
   getent passwd devuser1
   ```

### Expected Output

- `/etc/passwd` shows:  
  `devuser1:x:1001:1001::/home/devuser1:/bin/bash`

- `/etc/group` shows a matching group:  
  `devuser1:x:1001:`

- Home directory exists with ownership `devuser1:devuser1`.


## Exercise 2: Managing Groups and Secondary Group Membership

### Goal
Create shared departmental groups and assign users to them for collaborative access.

### Instructions

1. Create two groups for departments:
   ```bash
   groupadd finance
   groupadd engineering
   ```

2. Create two new users:
   ```bash
   useradd -m amar
   useradd -m yasser
   passwd amar
   passwd yasser
   ```

3. Add `amar` to the `finance` group and `yasser` to `engineering`:
   ```bash
   usermod -aG finance amar
   usermod -aG engineering yasser
   ```

4. Add both users to a shared `projects` group:
   ```bash
   groupadd projects
   usermod -aG projects amar
   usermod -aG projects yasser
   ```

5. Verify group memberships:
   ```bash
   groups amar
   groups yasser
   id amar
   id yasser
   ```

### Expected Output

- `amar` belongs to: `amar`, `finance`, `projects`
- `yasser` belongs to: `yasser`, `engineering`, `projects`

### Instructor Notes

- Always use `-aG` (append to groups) with `usermod`. Omitting `-a` **replaces** all secondary groups!
- In enterprises, departmental groups map to organizational units (e.g., HR, DevOps).
- Secondary groups are essential for shared directories (e.g., `/shared/projects` with `setgid`).


## Exercise 3: Configuring Password Aging and Hashing

### Goal
Enforce enterprise password policies using `chage` and configure secure password hashing.

### Instructions

1. View current password aging for `amar`:
   ```bash
   chage -l amar
   ```

2. Enforce the following policy:
   - Minimum password age: 1 day
   - Maximum password age: 90 days
   - Warning period: 7 days
   - Account lock after 180 days of inactivity

   Apply with:
   ```bash
   chage -m 1 -M 90 -W 7 -I 180 amar
   ```

3. Verify:
   ```bash
   chage -l amar
   ```

4. Check current password hashing algorithm:
   ```bash
   authselect current
   ```

5. Ensure system uses **SHA-512** (default in OL8). Confirm in `/etc/login.defs`:
   ```bash
   grep ENCRYPT_METHOD /etc/login.defs
   ```

   Expected output:
   ```
   ENCRYPT_METHOD SHA512
   ```

6. (Optional) Manually change hashing method if needed:
   ```bash
   authselect select sssd with-sudo --force
   # SHA-512 is standard; no change usually needed in OL8
   ```

### Expected Output

- `chage -l amar` shows updated min/max/warn/inactive values.
- `/etc/login.defs` confirms `SHA512`.

### Instructor Notes

- Password aging is critical for compliance (e.g., PCI-DSS, HIPAA).
- **Never** use MD5 or DES in production.
- Changes via `chage` affect `/etc/shadow`.
- `authselect` manages PAM stack—do not edit `/etc/pam.d/*` directly unless necessary.


## Exercise 4: Restricting the `su` Command

### Goal
Prevent unauthorized users from switching to root or other accounts using `su`.

### Instructions

1. Create a new group `wheel` (if not present):
   ```bash
   groupadd wheel
   ```

2. Add `amar` to the `wheel` group:
   ```bash
   usermod -aG wheel amar
   ```

3. Edit the PAM configuration for `su`:
   ```bash
   vi /etc/pam.d/su
   ```

4. Uncomment the following line (around line 6):
   ```
   auth            required        pam_wheel.so use_uid
   ```

5. Save and exit.

6. Test:
   - Log in as `yasser` and try `su -`
   - Log in as `amar` and try `su -`

### Expected Output

- `yasser` receives: `Permission denied`
- `amar` is prompted for root password and succeeds (if correct)

### Instructor Notes

- This enforces **least privilege**: only `wheel` members can `su` to root.
- In modern environments, `sudo` is preferred over `su`.
- Always test access controls in a non-production environment first.
- Ensure at least one user remains in `wheel` to avoid lockout.


## Exercise 5: Configuring Granular `sudo` Access

### Goal
Grant specific administrative privileges without full root access.

### Instructions

1. Ensure `sudo` is installed:
   ```bash
   rpm -q sudo || dnf install -y sudo
   ```

2. Create a new user `deployer`:
   ```bash
   useradd -m deployer
   passwd deployer
   ```

3. Allow `deployer` to restart the `httpd` service **without a password**:
   ```bash
   visudo
   ```

   Add the following line at the end:
   ```
   deployer ALL=(ALL) NOPASSWD: /bin/systemctl restart httpd
   ```

4. Test as `deployer`:
   ```bash
   sudo systemctl restart httpd
   ```

5. Now create a group `sysops` and allow all members to manage system services:
   ```bash
   groupadd sysops
   usermod -aG sysops yasser
   ```

   In `visudo`, add:
   ```
   %sysops ALL=(ALL) NOPASSWD: /bin/systemctl *
   ```

6. Test as `yasser`:
   ```bash
   sudo systemctl status firewalld
   ```

### Expected Output

- `deployer` can restart `httpd` without password.
- `yasser` can run any `systemctl` command without password.

### Instructor Notes

- **Always use `visudo`**—it checks syntax before saving.
- Avoid `ALL=(ALL) ALL` unless absolutely necessary.
- Use absolute paths in sudoers (e.g., `/bin/systemctl`, not `systemctl`).
- In enterprises, sudo rules are often managed via centralized tools (e.g., Ansible, LDAP).


## Exercise 6: Understanding and Editing Core Configuration Files

### Goal
Inspect and safely modify user/group configuration files.

### Instructions

1. Examine `/etc/passwd`:
   ```bash
   head -5 /etc/passwd
   ```

2. Examine `/etc/group`:
   ```bash
   tail -5 /etc/group
   ```

3. Examine `/etc/shadow` (note: only root can read):
   ```bash
   grep amar /etc/shadow
   ```

4. Manually create a user **without** `useradd` (for learning only):
   - Edit `/etc/passwd`:
     ```bash
     vipw
     ```
     Add:
     ```
     manualuser:x:2000:2000:Manual User:/home/manualuser:/bin/bash
     ```

   - Edit `/etc/group`:
     ```bash
     vigr
     ```
     Add:
     ```
     manualuser:x:2000:
     ```

   - Create home directory and set ownership:
     ```bash
     mkdir /home/manualuser
     cp /etc/skel/.* /home/manualuser/ 2>/dev/null || true
     chown -R manualuser:manualuser /home/manualuser
     chmod 700 /home/manualuser
     ```

   - Set password:
     ```bash
     passwd manualuser
     ```

5. Test login:
   ```bash
   su - manualuser
   ```

### Expected Output

- `manualuser` can log in with a home directory and shell.

### Instructor Notes

- **Never edit `/etc/passwd` or `/etc/group` directly in production**—use `useradd`, `groupadd`, etc.
- `vipw` and `vigr` lock files during editing to prevent corruption.


