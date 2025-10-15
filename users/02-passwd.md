# Linux `passwd` Command: All Use Cases

The `passwd` command is used to:

* Set or change user passwords
* Lock/unlock accounts
* Set password expiry policies
* Manage account aging

 

## Syntax

```bash
passwd [OPTIONS] [USERNAME]
```

 

## Basic Use Cases

### 1. Change Your Own Password

```bash
passwd
```

* Prompts for your **current password**, then **new password**.
* Validates strength and policy (depends on system config).

 

### 2. Set or Change Another User’s Password (as root)

```bash
sudo passwd username
```

* Does **not require current password**.
* Used by admins to set/reset user passwords.

Example:

```bash
sudo passwd alice
```

 

### 3. Lock a User Account

```bash
sudo passwd -l username
```

* Locks the user’s password (adds `!` to `/etc/shadow`).
* Prevents login using password (but not necessarily via SSH key).

Example:

```bash
sudo passwd -l bob
```

 

### 4. Unlock a User Account

```bash
sudo passwd -u username
```

* Unlocks an account that was previously locked with `-l`.

Example:

```bash
sudo passwd -u bob
```

 

## Advanced Use Cases: Password Expiration & Aging

You can control **password policies** per user.

### 5. Force Password Expiration Immediately

```bash
sudo passwd -e username
```

* Forces user to change password **on next login**.

Example:

```bash
sudo passwd -e alice
```

 

### 6. Disable Password Expiration

```bash
sudo passwd -x -1 username
```

* `-x -1` means password **never expires**.

 

### 7. Set Password Expiry Days

| Option    | Meaning                                                         |
|     |                       |
| `-x DAYS` | Max days before password must be changed                        |
| `-n DAYS` | Min days before password can be changed                         |
| `-w DAYS` | Warning days before password expires                            |
| `-i DAYS` | Inactive days after password expires before account is disabled |

#### Example: Set policy for user `bob`

```bash
sudo passwd -x 90 -n 7 -w 14 -i 30 bob
```

* Max: 90 days
* Min: 7 days
* Warn: 14 days before
* Inactive: 30 days after expiry

 

### 8. View Current Password Aging Info (with `chage`)

```bash
sudo chage -l username
```

Example:

```bash
sudo chage -l alice
```

 

### 9. Disable a Password (make account passwordless)

```bash
sudo passwd -d username
```

* Deletes the password hash from `/etc/shadow`
* User **won’t be able to log in using password**, but may still log in with SSH key or sudo if session is alive.

Example:

```bash
sudo passwd -d alice
```

 

### 10. Add Password to an Account That Has None

```bash
sudo passwd username
```

* If the account was created without a password (e.g., via `useradd`), this sets it for the first time.

 

## Password File Locations (For Reference)

| File          | Purpose                                |
| ------------- | -------------------------------------- |
| `/etc/shadow` | Stores password hashes and aging info  |
| `/etc/passwd` | Stores basic user info (not passwords) |

 

## Summary of All `passwd` Options

| Option        | Description                         |
| ------------- | ----------------------------------- |
| `passwd`      | Change your own password            |
| `passwd USER` | Change/set another user’s password  |
| `-l USER`     | Lock account                        |
| `-u USER`     | Unlock account                      |
| `-e USER`     | Expire password immediately         |
| `-d USER`     | Delete password (make passwordless) |
| `-x DAYS`     | Set max days password is valid      |
| `-n DAYS`     | Set min days before password change |
| `-w DAYS`     | Set warning days before expiration  |
| `-i DAYS`     | Set inactive days after expiration  |

 
