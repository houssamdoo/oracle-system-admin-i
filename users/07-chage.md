# Real-World `chage` Use Cases (With All Options)

These examples use all of `chage`'s main options:

| Option | Meaning                              |
| ------ | ------------------------------------ |
| `-l`   | List user password aging info        |
| `-d`   | Set last password change date        |
| `-E`   | Set account expiration date          |
| `-m`   | Minimum days before password change  |
| `-M`   | Maximum password validity period     |
| `-W`   | Warning days before password expires |
| `-I`   | Inactive days after password expiry  |


## Use Case 1: Enforce Regular Password Changes for All Staff

**Scenario**: For compliance reasons (e.g., ISO/PCI), users must change passwords every 90 days, with a 14-day warning, and a 7-day minimum between changes.

```bash
sudo chage -M 90 -W 14 -m 7  sarah
```

**Explanation**:

* Password expires in 90 days
* Warning starts 14 days before
* User can't change password more than once in 7 days

**Check:**

```bash
sudo chage -l  sarah
```


## Use Case 2: Set a Password Expiry Date for Temporary Accounts

**Scenario**: A contractor is given an account that should be active only until December 31, 2025.

```bash
sudo chage -E 2025-12-31 contractor
```

**Explanation**:

* After this date, the account will automatically be locked.

**Check:**

```bash
sudo chage -l contractor
```


## Use Case 3: Disable Inactive Accounts After Password Expiry

**Scenario**: After a password expires, if the user doesn’t log in within 10 days, the account should be disabled.

```bash
sudo chage -I 10 devuser
```

**Explanation**:

* If `devuser` doesn't log in and change their password within 10 days of expiration, the account becomes **inactive**.


## Use Case 4: Force Password Change on Next Login

**Scenario**: You want a user to set a new password the next time they log in — common after account creation or a reset.

```bash
sudo chage -d 0  sarah
```

**Explanation**:

* Sets the "last password change" to today → triggers mandatory password update at next login.


## Use Case 5: Review a User's Password Aging Info

**Scenario**: You want to check when a user's password will expire or when they last changed it.

```bash
sudo chage -l  sarah
```

Sample Output:

```
Last password change				 : Sep 14, 2025
Password expires					 : Dec 13, 2025
Password inactive					 : never
Account expires					 : never
Minimum number of days between password change	 : 7
Maximum number of days between password change	 : 90
Number of days of warning before password expires : 14
```



## Use Case 6: Reset Password Policy to No Expiration

**Scenario**: You want to disable password aging completely for a service account.

```bash
sudo chage -m 0 -M 99999 -I -1 -E -1 serviceuser
```

**Explanation**:

* No minimum days
* Password expires after 99999 days (effectively never)
* No account expiration (`-E -1`)
* No inactivity period (`-I -1`)


## Use Case 7: Setup a Full Policy for New Hires

**Scenario**: After onboarding a new employee, apply a standard password policy:

```bash
# Force password change on first login
sudo chage -d 0 newhire

# Enforce security policy
sudo chage -m 3 -M 90 -W 10 -I 7 -E 2026-12-31 newhire
```

**Explanation**:

* User must change password on first login
* Must wait 3 days before changing again
* Password expires every 90 days
* Gets 10-day warning before expiry
* Account disabled 7 days after password expiration
* Account expires at end of next year


## Script to Audit and Apply Policy for All Users

```bash
#!/bin/bash

for user in $(cut -f1 -d: /etc/passwd); do
    sudo chage -M 90 -W 14 -m 7 -I 10 "$user"
done
```

Use with caution — ensure you don’t affect service/system users (`nologin`, `false`, etc.).


## Summary: What Each Option Does

| Option    | Use Case                                              |
| --------- | ----------------------------------------------------- |
| `-l`      | View user’s current password aging settings           |
| `-d 0`    | Force password change on next login                   |
| `-M DAYS` | Set how long a password is valid                      |
| `-m DAYS` | Set how soon it can be changed again                  |
| `-W DAYS` | Set how many days before expiration to warn the user  |
| `-I DAYS` | Set how many days after expiry to disable the account |
| `-E DATE` | Set when the account will be fully disabled           |
