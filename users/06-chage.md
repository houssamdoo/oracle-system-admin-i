
# Linux `chage` Command: Complete Guide & Concepts

## What is `chage`?

The **`chage`** command (short for "**change age**") is used to **view and configure password aging and account expiry information** for a user.

It allows system administrators to enforce:

* Password expiration intervals
* Forced password changes
* Inactive account periods
* Account expiration dates


## Basic Syntax

```bash
sudo chage [options] username
```


## Common Use Cases

### 1. View Password Aging Information

```bash
sudo chage -l username
```

> This shows a user’s current password policy in a human-readable format.

Example:

```bash
sudo chage -l alice
```

Typical output:

```
Last password change				 : Sep 14, 2025
Password expires				 : Dec 13, 2025
Password inactive				 : never
Account expires					 : never
Minimum number of days between password change	 : 0
Maximum number of days between password change	 : 90
Number of days of warning before password expires : 7
```


## Modifying Password Aging Policies

### 2. Set Maximum Days Password is Valid

```bash
sudo chage -M DAYS username
```

* After this many days, the password expires.
* User will be prompted to change it at next login.

Example:

```bash
sudo chage -M 60 alice
```

> Alice must change her password every 60 days.


### 3. Set Minimum Days Between Password Changes

```bash
sudo chage -m DAYS username
```

* Prevents users from changing passwords too frequently (to cycle back to the old one).

Example:

```bash
sudo chage -m 2 alice
```

> Alice must wait 2 days between password changes.


### 4. Set Warning Period Before Password Expires

```bash
sudo chage -W DAYS username
```

* How many days before expiration the user gets a warning at login.

Example:

```bash
sudo chage -W 7 alice
```

> Alice will be warned for 7 days before her password expires.

---

### 5. Set Inactive Days After Password Expiry

```bash
sudo chage -I DAYS username
```

* After password expiration, this sets the number of **inactive days** before the account is **disabled**.

Example:

```bash
sudo chage -I 10 alice
```

> If Alice doesn’t change her password within 10 days after expiry, her account is locked.

---

### 6. Set Account Expiration Date

```bash
sudo chage -E YYYY-MM-DD username
```

* Sets the **absolute expiration date** for the account.

Example:

```bash
sudo chage -E 2025-12-31 alice
```

> Alice’s account will be disabled after December 31, 2025.

You can also use `-1` to set "never expires":

```bash
sudo chage -E -1 alice
```

---

### 7. Set Date of Last Password Change

```bash
sudo chage -d YYYY-MM-DD username
```

* Used to set or simulate a password change date.

Example:

```bash
sudo chage -d 2025-10-01 alice
```

> Makes the system behave as if Alice last changed her password on Oct 1, 2025.

Setting it to `0` forces password change at next login:

```bash
sudo chage -d 0 alice
```

## All `chage` Options Summary

| Option    | Description                                                    |
| --------- | -------------------------------------------------------------- |
| `-l`      | List user's aging information                                  |
| `-d DATE` | Set last password change date                                  |
| `-E DATE` | Set account expiration date                                    |
| `-m DAYS` | Set minimum days between password changes                      |
| `-M DAYS` | Set maximum days password is valid                             |
| `-I DAYS` | Set days of inactivity after password expires before disabling |
| `-W DAYS` | Set warning period before password expires                     |
| `-h`      | Show help message                                              |


## Real-World Examples

### Example 1: Enforce Strong Password Policy

```bash
sudo chage -M 90 -m 7 -W 14 alice
```

* Password valid for 90 days
* Can’t change again for 7 days
* Warning starts 14 days before expiration


### Example 2: Temporary Contractor Account

```bash
sudo useradd -m contractor
sudo passwd contractor
sudo chage -M 30 -E 2025-11-30 contractor
```

* Expires in 30 days
* Account disabled after Nov 30, 2025


### Example 3: Disable Inactive Account

```bash
sudo chage -I 15 alice
```

* If Alice doesn't log in or change her expired password within 15 days, account is disabled.


## Checking Values Programmatically

You can view raw values using:

```bash
sudo chage -l username
sudo grep '^username:' /etc/shadow
```

The `/etc/shadow` file has aging fields in this format:

```
username:encrypted_password:last_changed:min:max:warn:inactive:expire
```

## Practice Exercise

Try the following:

```bash
sudo useradd -m testuser
sudo passwd testuser
sudo chage -M 60 -m 3 -W 5 -I 10 -E 2025-12-31 testuser
sudo chage -l testuser
```


## Summary Table

| Goal                                | Command                        |
| ----------------------------------- | ------------------------------ |
| View password aging info            | `chage -l username`            |
| Set max password age                | `chage -M 60 username`         |
| Set min password age                | `chage -m 3 username`          |
| Set warning days                    | `chage -W 7 username`          |
| Set inactive days                   | `chage -I 10 username`         |
| Set account expiry date             | `chage -E 2025-12-31 username` |
| Force password change on next login | `chage -d 0 username`          |


