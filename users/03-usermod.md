# Linux `usermod` Command: Complete Guide & Use Cases

## Purpose of `usermod`

The `usermod` command is used to **modify an existing user account** in Linux. You can change a user's:

* Username
* Home directory
* UID/GID
* Login shell
* Groups
* Account expiry
* Lock status
* And more

 

## Basic Syntax

```bash
sudo usermod [OPTIONS] USERNAME
```

 

## Common Use Cases

### 1. Change Username

```bash
sudo usermod -l newname oldname
```

> Only changes the login name, **not the home directory**.

Example:

```bash
sudo usermod -l alice_new alice
```

 

### 2. Change User's Home Directory

```bash
sudo usermod -d /new/home/path username
```

To **move files** to the new location:

```bash
sudo usermod -d /new/home/path -m username
```

Example:

```bash
sudo usermod -d /home/alice_new -m alice
```

 

### 3. Change Login Shell

```bash
sudo usermod -s /bin/zsh username
```

Example:

```bash
sudo usermod -s /bin/zsh alice
```

 

### 4. Change User ID (UID)

```bash
sudo usermod -u 1050 username
```

Useful for aligning users across systems (e.g., in NFS or shared environments).

 

### 5. Change Primary Group

```bash
sudo usermod -g groupname username
```

Example:

```bash
sudo usermod -g developers alice
```

 

### 6. Add User to Supplementary Groups

```bash
sudo usermod -aG group1,group2 username
```

> Always use `-a` (append) with `-G` to avoid overwriting groups.

Example:

```bash
sudo usermod -aG sudo,docker alice
```

 

### 7. Set Account Expiry Date

```bash
sudo usermod -e YYYY-MM-DD username
```

Example:

```bash
sudo usermod -e 2025-12-31 alice
```

 

### 8. Disable Account on Password Expiry

```bash
sudo usermod -f 30 username
```

* Account will be **disabled 30 days** after password expiry.

 

### 9. Change User's Comment (Full Name or Info)

```bash
sudo usermod -c "Alice Smith - DevOps" alice
```

Check it with:

```bash
getent passwd alice
```

 

### 10. Lock or Unlock the User (via `passwd` instead)

Although `usermod` doesn't directly lock users, you can use `passwd`:

```bash
sudo passwd -l alice   # lock
sudo passwd -u alice   # unlock
```

 

## All Important `usermod` Options

| Option       | Description                                      |
|      |                  |
| `-l NEWNAME` | Change login name                                |
| `-d DIR`     | Change home directory                            |
| `-m`         | Move content to new home dir                     |
| `-s SHELL`   | Change login shell                               |
| `-u UID`     | Change UID                                       |
| `-g GROUP`   | Set primary group                                |
| `-G GROUPS`  | Set supplementary groups                         |
| `-a`         | Append to groups (used with `-G`)                |
| `-e DATE`    | Set account expiration date                      |
| `-f DAYS`    | Set inactivity days after password expires       |
| `-c COMMENT` | Change GECOS/comment field                       |
| `-L`         | Lock password (not commonly used — use `passwd`) |
| `-U`         | Unlock password (use `passwd`)                   |
| `-o`         | Allow duplicate UID (use with `-u`)              |

 

## Use Case Summary Table

| Use Case              | Command Example                           |
|         |              -- |
| Change username       | `usermod -l newname oldname`              |
| Change home directory | `usermod -d /new/home -m alice`           |
| Change shell          | `usermod -s /bin/zsh alice`               |
| Change UID            | `usermod -u 1050 alice`                   |
| Change primary group  | `usermod -g staff alice`                  |
| Add to groups         | `usermod -aG sudo,docker alice`           |
| Set expiry date       | `usermod -e 2025-12-31 alice`             |
| Set inactivity        | `usermod -f 30 alice`                     |
| Change comment        | `usermod -c "Alice Smith - DevOps" alice` |

 

## Practice Exercise

1. Create a user `testuser`
2. Change shell to `/bin/zsh`
3. Change home directory to `/srv/testuser`
4. Add to `sudo` and `docker`
5. Set expiration to `2025-12-31`

```bash
sudo useradd -m testuser
sudo usermod -s /bin/zsh testuser
sudo usermod -d /srv/testuser -m testuser
sudo usermod -aG wheel,sshd testuser
sudo usermod -e 2025-12-31 testuser
```

 

## Notes and Best Practices

* Always **backup** before changing UID/GID — it affects file ownership.
* When changing username or home, update other services/scripts that reference the old name.
* Use `-a` with `-G`, or it **overwrites** group memberships.

 

## Verification Commands

| Check            | Command                  |
|      - |          |
| View user info   | `getent passwd username` |
| View groups      | `groups username`        |
| View shadow info | `sudo chage -l username` |

 

## Summary

* `usermod` modifies **existing users**.
* Can change username, home, shell, UID, GID, groups, expiry, etc.
* Combine options for full updates.
* Use with care on production systems.

