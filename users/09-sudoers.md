# Sudoers Configuration Training Module: Oracle Linux 8

## Module Overview & Learning Objectives

### Concept Overview
The **sudo** (superuser do) mechanism provides controlled delegation of administrative privileges without sharing the **root** password, creating an essential **security layer** and **audit trail** . In enterprise environments, proper sudoers configuration balances operational efficiency with security compliance.

### Learning Objectives
After completing this module, you will be able to:
- Understand the **security benefits** of sudo versus direct root access
- Configure user and group **permissions** using visudo
- Implement **command-specific restrictions** for least privilege access
- Apply **enterprise best practices** for sudoers management
- **Monitor and audit** sudo usage effectively

## Core Concepts & Syntax

### sudo vs su: Security Comparison

| **Aspect**           | **sudo**                      | **su**                    |
| -------------------- | ----------------------------- | ------------------------- |
| **Authentication**   | User's own password           | root password             |
| **Audit Trail**      | Commands logged per user      | Only root activity logged |
| **Granular Control** | Command-specific restrictions | All-or-nothing access     |
| **Password Sharing** | Not required                  | Root password shared      |

### Key Configuration Files
- `/etc/sudoers` - Primary configuration file 
- `/etc/sudoers.d/` - Directory for supplemental configurations 

### Basic Syntax Structure
```bash
# User privilege specification
username  host=(runas_user:runas_group)  command 

# Group privilege specification  
%groupname  host=(runas_user:runas_group)  command 
```

## Configuration Examples & Use Cases

### Example 1: Basic User Privileges
```bash
# Allow user 'alice' to run dnf commands on all hosts
alice   ALL = /usr/bin/dnf 

# Allow user 'bob' full sudo access on all hosts  
bob     ALL=(ALL) ALL 
```

### Example 2: Command Aliases for Role-Based Access
```bash
# Define command categories
Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/yum 
Cmnd_Alias SERVICES = /sbin/service, /usr/bin/systemctl 

# Grant specific command categories to users
alice   ALL= SERVICES, SOFTWARE 
```

### Example 3: Group-Based Management
```bash
# Grant wheel group full administrative access
%wheel  ALL=(ALL) ALL 

# Add user to wheel group
sudo usermod -aG wheel bob 
```

### Example 4: Enhanced Security Configurations
```bash
# No-password requirement for specific commands (use cautiously)
%backup   ALL=(ALL) NOPASSWD: /usr/bin/rsync 

# Environment preservation for proxy settings
sudo -E curl https://www.example.com 
```

### Lab Scenario
You're administering an Oracle Linux 8 system with three user types:
- **Junior Admin** (user1): Needs service management privileges only
- **Package Manager** (user2): Needs software installation privileges only  
- **Senior Admin** (user3): Full administrative access via wheel group


#### Step 1: Create User Accounts
```bash
# Create users
sudo useradd user1
sudo useradd user2  
sudo useradd user3

# Set passwords
sudo passwd user1
sudo passwd user2
sudo passwd user3
```

#### Step 2: Configure Service Management Privileges
```bash
# Create dedicated configuration file
sudo visudo -f /etc/sudoers.d/service-management

# Add service management permissions
user1   ALL = SERVICES 
```

#### Step 3: Configure Package Management Privileges  
```bash
# Create package management configuration
sudo visudo -f /etc/sudoers.d/package-management

# Add package commands
user2   ALL = SOFTWARE 
```

#### Step 4: Configure Wheel Group Access
```bash
# Ensure wheel group is enabled in sudoers
sudo visudo

# Verify this line is uncommented:
%wheel  ALL=(ALL) ALL 

# Add user3 to wheel group
sudo usermod -aG wheel user3 
```

#### Step 5: Testing and Validation
```bash
# Test service management privileges (as user1)
sudo systemctl status sshd

# Test package management privileges (as user2)  
sudo dnf update httpd -y

# Test full administrative privileges (as user3)
sudo useradd testuser
```

## USE CASE 2
### Use Case 1: Database Administration Team
**Requirements**: Oracle database admins need to manage database processes but not system services.

**Solution**:
```bash
Cmnd_Alias DB_COMMANDS = /bin/su - oracle, /usr/bin/systemctl start oracle*, /usr/bin/systemctl stop oracle*

%dba   ALL = DB_COMMANDS
```

### Use Case 2: Backup Administration  
**Requirements**: Backup operators need filesystem access without full root privileges.

**Solution**:
```bash
Cmnd_Alias BACKUP_CMDS = /usr/bin/rsync, /bin/tar, /sbin/lvcreate

%backup   ALL=(ALL) NOPASSWD: BACKUP_CMDS 
```

### Use Case 3: Web Administrators
**Requirements**: Web admins need to manage web services and configuration files.

**Solution**:
```bash
Cmnd_Alias WEB_CMDS = /usr/bin/systemctl reload nginx, /usr/bin/systemctl restart nginx, /bin/cp /etc/nginx/*

%webadmins   ALL = WEB_CMDS
```

## Security Best Practices & Monitoring

### Security Hardening Recommendations

1. **Use visudo Exclusively**: Always use `visudo` for edits to prevent syntax errors and file corruption 
2. **Leverage sudoers.d Directory**: Place custom configurations in `/etc/sudoers.d/` for upgrade-safe management 
3. **Implement Principle of Least Privilege**: Grant only necessary commands, not full ALL access 
4. **Enable Comprehensive Logging**: Monitor sudo usage through system logs 

### Audit Configuration
```bash
# Configure audit rules for sudoers changes
sudo printf '-w /etc/sudoers -p wa -k scope\n-w /etc/sudoers.d -p wa -k scope\n' >> /etc/audit/rules.d/50-scope.rules 

# Load audit rules
sudo augenrules --load 
```

### Log Monitoring
```bash
# Monitor sudo access attempts
sudo grep sudo /var/log/secure 

# Check authentication logs
sudo grep 'NOT in sudoers' /var/log/secure 
```

