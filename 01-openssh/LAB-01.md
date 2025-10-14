# Lab-02: Key-Based Authentication on Linux

**Objective:** To understand, configure, and troubleshoot key-based authentication for SSH on Linux systems.

**Prerequisites:**

*   Two Linux virtual machines (VMs) or cloud instances. For simplicity, we'll refer to them as `client_vm` and `server_vm`.
    *   Both VMs should have SSH installed and running (`sshd` service).
    *   Basic network connectivity between the two VMs.
    *   Sudo privileges on both VMs.
*   A terminal or SSH client on your local machine to connect to `client_vm`.

---

#### **Part 1: Initial Setup and Verification**

**Step 1: Verify SSH Connectivity (from your local machine to `client_vm` and then from `client_vm` to `server_vm`)**

1.  **Connect to `client_vm` from your local machine:**
    ```bash
    ssh username@client_vm_ip_address
    ```
    *   Enter your password when prompted. If this is your first time connecting, you might be asked to confirm the host's authenticity. Type `yes`.
2.  **From `client_vm`, try connecting to `server_vm`:**
    ```bash
    ssh username@server_vm_ip_address
    ```
    *   Again, enter your password and confirm host authenticity if prompted.

**Expected Outcome:** You should be able to successfully connect to both VMs using password authentication. This confirms basic network and SSH functionality.

---

#### **Part 2: Generating SSH Key Pair on `client_vm`**

Key-based authentication relies on a pair of cryptographic keys: a private key (kept secret on the client) and a public key (shared with the server).

**Step 1: Log in to `client_vm` (if not already logged in).**

**Step 2: Generate the SSH key pair:**
```bash
ssh-keygen -t rsa -b 4096
```
*   **`-t rsa`**: Specifies the type of key to create (RSA is a common and secure choice).
*   **`-b 4096`**: Specifies the number of bits in the key. 4096 bits is a strong recommendation for security.
*   **Output:**
    ```
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/username/.ssh/id_rsa):
    ```
    *   **Press Enter** to accept the default location (`/home/username/.ssh/id_rsa`). This is the standard and recommended path.
    ```
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    ```
    *   **Enter a strong passphrase** (recommended for security) or **press Enter twice** for no passphrase (less secure, but sometimes used in automated scripts). For this lab, let's **use a passphrase** to experience it. Remember it!

**Expected Outcome:** Two files will be created in `~/.ssh/`:
*   `id_rsa`: Your private key. **DO NOT SHARE THIS FILE.**
*   `id_rsa.pub`: Your public key. This is safe to share.

**Step 3: Examine the generated files.**
```bash
ls -l ~/.ssh/
```
```bash
cat ~/.ssh/id_rsa.pub
```
*   The `id_rsa.pub` file contains a long string starting with `ssh-rsa` followed by your key and a comment (usually your username@hostname).

---

#### **Part 3: Copying the Public Key to `server_vm`**

Now we need to place the public key from `client_vm` onto `server_vm` in the correct location.

**Step 1: Use `ssh-copy-id` (Recommended and Easiest Method)**

While still on `client_vm`:
```bash
ssh-copy-id username@server_vm_ip_address
```
*   You will be prompted for the password of `username` on `server_vm`.
*   You might also be asked if you want to continue connecting (type `yes`).

**Expected Output:**
```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/username/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
username@server_vm_ip_address's password:
```
*   After entering the password, it will report:
    ```
    Number of key(s) added: 1

    Now try logging into the machine, with:   "ssh 'username@server_vm_ip_address'"
    and check to make sure that only the key(s) you wanted were added.
    ```

**Step 2: (Alternative) Manual Public Key Copy (Understand the mechanism)**

*   If `ssh-copy-id` is not available or you want to understand the manual process:
    1.  **Copy the content of `id_rsa.pub` from `client_vm`:**
        ```bash
        cat ~/.ssh/id_rsa.pub
        ```
        *   Copy the *entire* output string.
    2.  **Connect to `server_vm` using password authentication:**
        ```bash
        ssh username@server_vm_ip_address
        ```
    3.  **On `server_vm`, create the `.ssh` directory if it doesn't exist and set correct permissions:**
        ```bash
        mkdir -p ~/.ssh
        chmod 700 ~/.ssh
        ```
    4.  **Append the public key to `~/.ssh/authorized_keys` file:**
        ```bash
        echo "PASTE_YOUR_PUBLIC_KEY_STRING_HERE" >> ~/.ssh/authorized_keys
        ```
        *   **Important:** Replace `"PASTE_YOUR_PUBLIC_KEY_STRING_HERE"` with the actual public key you copied.
    5.  **Set correct permissions for `authorized_keys`:**
        ```bash
        chmod 600 ~/.ssh/authorized_keys
        ```
    6.  **Exit from `server_vm`:**
        ```bash
        exit
        ```

**Expected Outcome:** The public key `id_rsa.pub` from `client_vm` is now appended to `~/.ssh/authorized_keys` on `server_vm`.

---

#### **Part 4: Testing Key-Based Authentication**

**Step 1: From `client_vm`, try to connect to `server_vm` again:**
```bash
ssh username@server_vm_ip_address
```
*   This time, instead of being prompted for `username@server_vm_ip_address`'s password, you should be prompted for your **private key passphrase** (if you set one).
*   Enter the passphrase you created in Part 2, Step 2.

**Expected Outcome:** You should successfully log in to `server_vm` without entering the user's password. This confirms key-based authentication is working.

---

#### **Part 5: Enhancing Security - Disabling Password Authentication**

Once key-based authentication is working reliably, it's highly recommended to disable password authentication on the server for improved security. This prevents brute-force attacks against user passwords.

**Step 1: Log in to `server_vm` (using key-based authentication from `client_vm` or directly if you can).**

**Step 2: Edit the SSH daemon configuration file:**
```bash
sudo nano /etc/ssh/sshd_config
```
*   Find the following lines and modify them as follows. If they are commented out (start with `#`), uncomment them and change their values:
    ```
    PasswordAuthentication no
    ChallengeResponseAuthentication no
    #PermitRootLogin prohibit-password   # Good practice, ensures root can't log in with password
    ```
    *   **Note on `PermitRootLogin`**: While `prohibit-password` allows key-based login for root (if configured), setting it to `no` is even more secure, forcing users to log in with a regular user and then `sudo`. For this lab, `prohibit-password` is fine.

**Step 3: Save the file and exit the editor.**

**Step 4: Restart the SSH service:**
```bash
sudo systemctl restart sshd
```

**Step 5: Test the change.**
*   **From `client_vm`**, try to connect to `server_vm` using SSH. You should still be able to connect using your key and passphrase.
*   **Crucially, try to connect from a *different* machine or directly from your local machine to `server_vm` using *password authentication*:**
    ```bash
    ssh username@server_vm_ip_address
    ```
    *   This connection attempt should now **fail** with a "Permission denied (publickey)" or similar message, indicating that password authentication is no longer accepted.

**Expected Outcome:** Only key-based authentication should be allowed for `server_vm`.

---

#### **Part 6: SSH Agent (For Convenience and Security with Passphrases)**

Entering your passphrase repeatedly can be tedious. The SSH agent holds your decrypted private keys in memory for the duration of your session, so you only need to enter the passphrase once.

**Step 1: Log in to `client_vm`.**

**Step 2: Start the SSH agent (if not already running):**
*   Often, the SSH agent is started automatically by your desktop environment or terminal. You can check:
    ```bash
    echo "$SSH_AGENT_PID"
    ```
    *   If it returns a number, it's running. If it's empty, you might need to start it.
    *   **To start it manually:**
        ```bash
        eval "$(ssh-agent -s)"
        ```
        *   This will output something like: `Agent pid <PID>`

**Step 3: Add your private key to the agent:**
```bash
ssh-add ~/.ssh/id_rsa
```
*   You will be prompted for your private key's passphrase. Enter it.

**Expected Output:**
```
Identity added: /home/username/.ssh/id_rsa (/home/username/.ssh/id_rsa)
```

**Step 4: Test connectivity again.**
```bash
ssh username@server_vm_ip_address
```
*   This time, you should be logged in directly to `server_vm` **without being asked for your passphrase**. The SSH agent is providing the key.

**Step 5: (Optional) Listing keys in the agent:**
```bash
ssh-add -l
```

**Step 6: (Optional) Removing keys from the agent:**
```bash
ssh-add -D
```
*   This will clear all identities from the agent. The agent will remain running, but you'll have to add keys again.

**Expected Outcome:** Seamless key-based login after adding the key to the agent.

---

#### **Part 7: Troubleshooting and Common Issues**

1.  **"Permission denied (publickey, password)." when using keys:**
    *   **Check `~/.ssh` directory permissions on `server_vm`:**
        *   `~/.ssh` should be `drwx------` (700).
        *   `~/.ssh/authorized_keys` should be `-rw-------` (600).
        *   Your home directory `~` should **not** be world-writable (`chmod 755 ~`).
    *   **Check `sshd_config` on `server_vm`:**
        *   `PubkeyAuthentication yes` (usually default, but verify).
        *   `AuthorizedKeysFile .ssh/authorized_keys` (usually default, but verify).
    *   **Check key ownership:** Ensure `.ssh` and `authorized_keys` are owned by the connecting user on `server_vm`.
    *   **Check SELinux/AppArmor:** If enabled, these might block SSH access. Temporarily disable them (`sudo setenforce 0` for SELinux) to test, then investigate specific policies.
    *   **Verbose SSH output:** `ssh -v username@server_vm_ip_address` can give clues.

2.  **"Agent forwarding failed: Permission denied"**
    *   This typically means agent forwarding is not enabled or working correctly.
    *   Ensure `AllowAgentForwarding yes` in `sshd_config` on the server you are connecting *to*.
    *   Ensure you are using `ssh -A` to forward the agent when connecting.

3.  **Forgotten Passphrase:**
    *   You cannot recover a forgotten passphrase. You will need to generate a new key pair and copy the new public key to all servers.

4.  **`ssh-copy-id` fails:**
    *   Ensure you can connect via password authentication first. `ssh-copy-id` uses password auth initially.
    *   Permissions on the target user's home directory or `.ssh` can prevent `ssh-copy-id` from working.
