# `useradd` Command

## Objectives

By the end of this course, you will be able to:

* Understand how user accounts work in Linux.
* Use the `useradd` command to create users.
* Understand all common `useradd` options.
* Set user passwords and manage user properties.


## 1 - Introduction to User Management

### What is a User in Linux?

A **user** is an identity used to log into a Linux system. Each user has:

* A **username**
* A **user ID (UID)**
* A **home directory**
* A **default shell**
* A **group ID (GID)**
* Configuration files in their home directory

### Key Files Involved

* `/etc/passwd` – user info
* `/etc/shadow` – passwords
* `/etc/group` – group info
* `/etc/default/useradd` – default values for user creation
* `/etc/login.defs` – user account configuration defaults


## 2 - Basic User Creation with `useradd`

### Basic Syntax

```bash
sudo useradd [options] username
```

### Example:

```bash
sudo useradd oracle
```

This creates a user called `oracle` with default options.

>Note: By default, **no password is set** and **no home directory** is created on some systems (like Debian-based distros). You need to add more options for full configuration.

Example:
```bash
sudo useradd -m -s /bin/bash 
```

## 3 - Common `useradd` Options Explained

Below are **common and important options** used with `useradd`.

| Option          | Description                                | Example                             |
| --------------- | ------------------------------------------ | ----------------------------------- |
| `-m`            | Create a home directory                    | `useradd -m oracle`                 |
| `-d DIR`        | Specify home directory                     | `useradd -m -d /custom/home oracle` |
| `-s SHELL`      | Set login shell                            | `useradd -s /bin/bash oracle`       |
| `-g GROUP`      | Set initial group                          | `useradd -g users oracle`           |
| `-G GROUPS`     | Add to supplementary groups                | `useradd -G sudo,adm oracle`        |
| `-u UID`        | Set custom UID                             | `useradd -u 1501 oracle`            |
| `-c COMMENT`    | Add description (GECOS)                    | `useradd -c "Oracle Oracle" oracle` |
| `-e YYYY-MM-DD` | Set account expiration date                | `useradd -e 2025-12-31 oracle`      |
| `-f DAYS`       | Set password inactivity (after expiration) | `useradd -f 10 oracle`              |
| `-r`            | Create a system account                    | `useradd -r sysuser`                |
| `-M`            | Don’t create home directory                | `useradd -M oracle`                 |
| `-N`            | Don’t create a user group                  | `useradd -N oracle`                 |
| `-o`            | Allow duplicate UID (with `-u`)            | `useradd -o -u 1000 duplicateuser`  |
| `-Z`            | Set SELinux user for the new account       | Depends on SELinux context          |

---

## 4 - Examples and Use Cases

### Example 1: Create a typical user with all defaults

```bash
sudo useradd -m -s /bin/bash oracle
sudo passwd oracle
```

### Example 2: Create user with custom home and expiry

```bash
sudo useradd -m -d /data/users/oracle -s /bin/bash -e 2025-12-31 -c "Data Analyst" oracle
```

### Example 3: Add user to multiple groups

```bash
sudo useradd -m -G sudo,developers -s /bin/bash bob
```

---

## 5 - Verifying Users

### View User Info:

```bash
getent passwd oracle
```

### Check Groups:

```bash
groups oracle
```

### View Home Directory Contents:

```bash
ls -la /home/oracle
```

---

## 6 - Common Mistakes

| Mistake           | Solution                                                     |
| ----------------- | ------------------------------------------------------------ |
| Home not created  | Use `-m` option                                              |
| Can't log in      | Ensure shell is valid (`/bin/bash`, not `/usr/sbin/nologin`) |
| User not in group | Use `-G` or `usermod -aG`                                    |

---

## 7 - Related Commands

| Command       | Purpose                     |
| ------------- | --------------------------- |
| `passwd USER` | Set or change user password |
| `usermod`     | Modify existing user        |
| `userdel`     | Delete user                 |
| `id USER`     | Show UID, GID, groups       |
| `whoami`      | Show current logged-in user |
