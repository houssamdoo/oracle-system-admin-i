# SSH Access Control: Comprehensive Lab Guide

This lab guide provides a comprehensive overview of SSH access control, focusing on the configuration and management of SSH keys, user permissions, and security best practices.

# Lab Overview

This lab provides hands-on experience with SSH access control mechanisms, from basic key authentication to advanced restrictions and monitoring.

In this lab, you will learn how to:

- Generate and manage SSH key pairs.
- Configure SSH server settings for enhanced security.
- Implement user access controls using SSH keys.
- Troubleshoot common SSH access issues.
- Understand best practices for SSH security.

## Lab 1: Basic SSH Access Setup & Key Management

### Objective

Set up foundational SSH access with key-based authentication and understand the components.

### Prerequisites

- Two Linux machines (or VMs) - one client, one server
- SSH server installed on the target machine
- Root/sudo access on both systems

### Exercises

1.1. **Initial SSH Server Configuration**:

- On the server machine, install the OpenSSH server if not already installed:
  
     ```bash
        # On the SSH server:
        sudo systemctl status sshd
        sudo ss -tlnp | grep :22
        # Examine default configuration
        #sudo cat /etc/ssh/sshd_config | grep -v "^#" | grep -v "^$"
        grep -vE '^\s*#|^$' /etc/ssh/sshd_config

     ```

1.2. **Key Pair Generation & Deployment**:

- On the client machine, generate an SSH key pair:
  
     ```bash
        # On the CLIENT machine:
        # Generate ED25519 key pair (modern, secure)
        ssh-keygen -t ed25519 -C "oracle@client" -f ~/.ssh/lab_key

        # Generate RSA key pair (for compatibility)
        ssh-keygen -t rsa -b 4096 -C "oracle@client" -f ~/.ssh/lab_rsa_key

        # View the generated keys
        ls -la ~/.ssh/*.pub
        cat ~/.ssh/lab_rsa_key.pub

        # Copy keys to server using different methods:

        # Method 1: ssh-copy-id (easiest)
        ssh-copy-id -i ~/.ssh/lab_rsa_key.pub username@server-ip

        # Method 2: Manual copy
        cat ~/.ssh/lab_rsa_key.pub | ssh username@server-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

        # Method 3: SCP method
        scp ~/.ssh/lab_rsa_key.pub username@server-ip:~/temp_key.pub
        ssh username@server-ip "mkdir -p ~/.ssh && cat ~/temp_key.pub >> ~/.ssh/authorized_keys && rm ~/temp_key.pub"
     ```

1.3. **Verify Key-Based Authentication**:

- Test SSH access using the deployed key:
  
     ```bash
        # Test connection with specific key
        ssh -i ~/.ssh/lab_rsa_key username@server-ip

        # Check server logs during connection
        # On SERVER:
        sudo tail -f /var/log/auth.log
        # OR
        sudo tail -f /var/log/secure
     ```

1.4. **Key File Permissions Security**:

- Ensure proper permissions on key files:
  
     ```bash
        # On CLIENT - Set proper permissions
        chmod 700 ~/.ssh
        chmod 600 ~/.ssh/lab_key
        chmod 644 ~/.ssh/lab_rsa_key.pub
        chmod 644 ~/.ssh/authorized_keys  # On server

        # On SERVER - Verify authorized_keys permissions
        chmod 600 ~/.ssh/authorized_keys
        chown $USER:$USER ~/.ssh/authorized_keys
     ```

---

## Lab 2: Server-Wide Access Control

### Objective

Configure system-wide SSH access restrictions using sshd_config.

### Exercises

2.1. **Backup and Analyze Current Config**:

- Backup the existing SSH configuration file and review its contents:
  
    ```bash
        # On SERVER:
        sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
        sudo grep -v "^#" /etc/ssh/sshd_config | grep -v "^$"
    ```

2.2. **Implement Access Restrictions (User-Based Access Control)**:

- Modify `sshd_config` to restrict access:
  
    ```bash
        # Edit SSH server configuration
        sudo vi /etc/ssh/sshd_config

        # Add these lines for user-based control:
        AllowUsers belkacem omar aicha@192.168.1.100
        DenyUsers mokhtar yazid
        # AllowGroups ssh-users admin-team
        # DenyGroups restricted-users

        # Explanation:
        # - AllowUsers: Only these users can SSH
        # - user@host: User only from specific IP
        # - DenyUsers: Explicitly block these users
        # - *Groups: Control by group membership
    ```

2.3. **Implement Access Restrictions (Network-Based Restrictions)**:

- Modify `sshd_config` to restrict access:
  
    ```bash
        # Add to /etc/ssh/sshd_config:
        # Restrict by IP/CIDR
        ListenAddress 0.0.0.0:22  # Or specific IP
        # Allow from specific networks only
        Match Address 192.168.1.0/24,10.0.0.0/8
            AllowUsers alice bob charlie
    ```

2.4. **Authentication Method Control**:

- Modify `sshd_config` to restrict access:
  
    ```bash
        # Control authentication methods
        PasswordAuthentication no              # Disable password auth
        PubkeyAuthentication yes               # Enable key auth
        PermitEmptyPasswords no
        ChallengeResponseAuthentication no
        KerberosAuthentication no
        GSSAPIAuthentication no

        # Restrict key options globally
        PermitRootLogin no                     # Critical security setting
        PermitTTY yes
        X11Forwarding no                       # Disable if not needed
    ```

2.5. **Apply and Test Configuration**:
- Restart the SSH service and test the new configuration:
    ```bash
        # Test configuration syntax
        sudo sshd -t

        # If syntax OK, restart service
        sudo systemctl restart sshd

        # Test from CLIENT:
        ssh -i ~/.ssh/lab_rsa_key allowed-user@server-ip    # Should work
        ssh denied-user@server-ip                       # Should be rejected
        ssh -i ~/.ssh/lab_rsa_key oracle@15.15.15.12

        # Monitor logs during testing:
        sudo tail -f /var/log/auth.log | grep sshd                     # Disable if not needed
    ```

---

## Lab 3: Advanced Key-Based Restrictions

### Objective
Implement granular control using advanced `authorized_keys` options.

### Exercises

3.1. **Key Command Restrictions**:

- Edit the `authorized_keys` file to restrict commands for a specific key:
  
    ```bash
        # On SERVER - Edit authorized_keys
        nano ~/.ssh/authorized_keys

        # Add command restrictions
        command="restricted_command" ssh-rsa AAAAB3... user@host
    ```

- Example:

    ```bash
        # On SERVER, edit ~/.ssh/authorized_keys:

        # Key can only run specific command
        command="/usr/bin/rbash" ssh-ed25519 AAA... lab-user@client

        # Key restricted to port forwarding only
        restrict,port-forwarding ssh-rsa AAA... lab-user-rsa@client

        # Key with source IP restriction
        from="192.168.1.50",command="/opt/scripts/backup.sh" ssh-ed25519 AAA...

        # Key with expiry date
        expiry-time="20241231",ssh-ed25519 AAA...
    ```
3.2. **Complex Key Restrictions**:

- Combine multiple restrictions on a single key:

    ```bash
        # Multiple restrictions on single key
        restrict,command="/usr/bin/rbash",from="192.168.1.0/24",no-agent-forwarding,no-port-forwarding,no-pty ssh-ed25519 AAA...

        # Time-based access (only during business hours)
        command="/bin/echo 'Access only 9AM-5PM'",no-pty,permitopen="192.168.1.10:80" ssh-rsa AAA...
    ```

3.3. **Create Restricted Service Accounts**:

- Create a user with limited shell access and restrict their SSH key usage:

    ```bash
        # On SERVER:
        # Create user with restricted shell
        sudo useradd -s /usr/bin/rbash -m backup-user
        sudo mkdir -p /home/backup-user/bin
        sudo ln -s /usr/bin/ls /home/backup-user/bin/ls
        sudo ln -s /usr/bin/cat /home/backup-user/bin/cat

        # Set up restricted authorized_keys
        sudo su - backup-user
        mkdir .ssh
        echo 'command="/home/backup-user/bin/ls /backups"' ssh-ed25519 AAA... > .ssh/authorized_keys
        chmod 600 .ssh/authorized_keys
        exit
    ```

3.4. **Enforce Key Expiration**:

- Implement key expiration by using `authorized_keys` options:

    ```bash
        # On SERVER - Edit authorized_keys
        nano ~/.ssh/authorized_keys

        # Add expiration
        expire="2025-12-31" ssh-rsa AAAAB3... user@host
    ```

3.5. **Testing and Validation**:

- Test the new restrictions by attempting to SSH with the restricted key:

    ```bash
        # On CLIENT, test restricted access:
        ssh -i ~/.ssh/lab_key backup-user@server-ip
        # Should only be able to run the specified command
    ```
