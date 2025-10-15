# Oracle Linux 8 User and Group Administration Lab Guide

## Exercise 1: User Account Creation and Management

### Goal
Create and manage basic user accounts with various attributes and configurations.

### Instructions

1. **Create basic user accounts:**
```bash
sudo useradd -c "Youcef Yazid - Developer" youcef
sudo useradd -c "Sarah Amina - DBA" sarah
sudo useradd -c "IT Support Team" itsupport
```

2. **Set passwords for the new users:**
```bash
echo "password123" | sudo passwd --stdin youcef
echo "password123" | sudo passwd --stdin sarah
echo "password123" | sudo passwd --stdin itsupport
```

3. **Create users with specific UIDs and home directories:**
```bash
sudo useradd -u 1501 -c "Database Administrator" -d /home/dba -s /bin/bash dba
sudo useradd -u 1601 -c "Web Developer" -d /home/webdev -s /bin/bash webdev
```

4. **Modify existing user properties:**
```bash
sudo usermod -c "Senior DBA" sarah
sudo usermod -s /bin/sh youcef
sudo usermod -d /home/youcef -m youcef
```

### Validation
```bash
# Verify user creation
id youcef
id sarah
id dba

# Check user properties
grep youcef /etc/passwd
grep sarah /etc/passwd

# Verify home directories
ls -la /home/
```

### Instructor Notes
- **Common Pitfall:** Forgetting to set passwords for new users, which leaves accounts locked
- **Real-world Usage:** Always use descriptive comments (`-c`) for user accountability
- **Security Note:** In production, use interactive `passwd` command instead of stdin for better security


## Exercise 2: Group Administration and User Assignment

### Goal
Create and manage groups, assign users to multiple groups, and understand group membership.

### Instructions

1. **Create departmental groups:**
```bash
sudo groupadd -g 2001 developers
sudo groupadd -g 2002 dba
sudo groupadd -g 2003 support
sudo groupadd -g 2004 managers
```

2. **Assign users to primary groups:**
```bash
sudo usermod -g developers youcef
sudo usermod -g dba sarah
sudo usermod -g support itsupport
```

3. **Add users to secondary groups:**
```bash
sudo usermod -aG dba,managers youcef
sudo usermod -aG developers,managers sarah
sudo usermod -aG developers itsupport
```

4. **Create a shared group for project collaboration:**
```bash
sudo groupadd project-alpha
sudo usermod -aG project-alpha youcef
sudo usermod -aG project-alpha sarah
```

### Validation
```bash
# Check group membership
groups youcef
groups sarah
id youcef

# Verify group creation
getent group developers
getent group project-alpha

# List all groups
getent group | tail -10
```

### Instructor Notes
- **Key Concept:** Primary group vs. supplementary groups
- **Common Pitfall:** Using `-G` without `-a` removes existing supplementary groups
- **Enterprise Context:** Groups often map to departments, projects, or access levels


## Exercise 3: Implementing User Private Groups (UPG)

### Goal
Understand and implement User Private Groups for enhanced file security and management.

### Instructions

1. **Create users with UPG scheme:**
```bash
sudo useradd -m -U devuser1
sudo useradd -m -U devuser2
```

2. **Create a shared directory with proper permissions:**
```bash
sudo mkdir -p /opt/shared_project
sudo groupadd project_team
sudo usermod -aG project_team devuser1
sudo usermod -aG project_team devuser2
```

3. **Configure shared directory permissions:**
```bash
sudo chgrp project_team /opt/shared_project
sudo chmod 2775 /opt/shared_project  # SETGID bit ensures files inherit group
ls -ld /opt/shared_project
```

4. **Test UPG functionality:**
```bash
# Switch to devuser1 and create a file
sudo su - devuser1
touch /opt/shared_project/file_from_devuser1
ls -l /opt/shared_project/file_from_devuser1
exit

# Switch to devuser2 and verify they can modify the file
sudo su - devuser2
echo "test" >> /opt/shared_project/file_from_devuser1
ls -l /opt/shared_project/file_from_devuser1
exit
```

### Validation
```bash
# Check UPG implementation
grep devuser1 /etc/passwd
grep devuser1 /etc/group

# Verify SETGID bit
ls -ld /opt/shared_project | grep rwxrwsr-x

# Test file creation and group ownership
sudo -u devuser1 touch /opt/shared_project/test_file
ls -l /opt/shared_project/test_file
```

### Instructor Notes
- **Key Concept:** UPG creates a personal group for each user with same name as username
- **Enterprise Benefit:** Simplifies permission management in collaborative environments
- **SETGID Importance:** Critical for ensuring files inherit parent directory's group


## Exercise 4: Password Policy and Security Configuration

### Goal
Implement enterprise password policies including aging, complexity, and hashing algorithms.

### Instructions

1. **Configure password aging for existing users:**
```bash
sudo chage -M 90 -m 7 -W 14 youcef
sudo chage -M 90 -m 7 -W 14 sarah
sudo chage -I 30 youcef  # Account inactive lock after 30 days
```

2. **View password aging information:**
```bash
sudo chage -l youcef
sudo chage -l sarah
```

## Final Challenge - Departmental Environment Setup

### Goal
Combine all learned skills to create a complete departmental environment with users, groups, permissions, and security configurations.

### Scenario
Set up the "Finance Department" environment with the following requirements:
- 3 user roles: Finance Managers, Finance Analysts, Finance Support
- Shared directory structure with appropriate permissions
- Role-based sudo access
- Password and security policies
- Restricted su access

### Instructions

1. **Create finance groups and users:**
```bash
# Create finance groups
sudo groupadd -g 3001 finance_mgrs
sudo groupadd -g 3002 finance_analysts  
sudo groupadd -g 3003 finance_support
sudo groupadd -g 3004 finance_all

# Create finance users with UPG
sudo useradd -m -U -c "Finance Manager 1" -G finance_mgrs,finance_all fmgr1
sudo useradd -m -U -c "Finance Analyst 1" -G finance_analysts,finance_all fanalyst1
sudo useradd -m -U -c "Finance Support 1" -G finance_support,finance_all fsupport1

# Set passwords
echo "FinMgr123!" | sudo passwd --stdin fmgr1
echo "FinAnalyst123!" | sudo passwd --stdin fanalyst1  
echo "FinSupport123!" | sudo passwd --stdin fsupport1
```

2. **Create directory structure with permissions:**
```bash
# Create finance directory structure
sudo mkdir -p /finance/{reports,shared,sensitive,archive}
sudo chgrp finance_all /finance
sudo chmod 2775 /finance  # SETGID for inheritance

# Set specific directory permissions
sudo chgrp finance_mgrs /finance/sensitive
sudo chmod 2770 /finance/sensitive  # Managers only + SETGID

sudo chgrp finance_analysts /finance/reports
sudo chmod 2775 /finance/reports  # Analysts can write, all can read

sudo chgrp finance_all /finance/shared
sudo chmod 2777 /finance/shared  # All finance can read/write

sudo chgrp finance_support /finance/archive  
sudo chmod 2775 /finance/archive  # Support manages archives
```

3. **Configure finance-specific sudo rules:**
```bash
sudo cat > /tmp/10_finance << 'EOF'
# Finance department sudo rules

# Finance Managers - broad financial system access
%finance_mgrs ALL=(ALL) /bin/systemctl status finapp*, \
                        /usr/bin/tail -f /var/log/finapp*, \
                        /usr/bin/cat /finance/sensitive/*, \
                        /usr/bin/rm /finance/archive/*

# Finance Analysts - reporting tools access  
%finance_analysts ALL=(ALL) /usr/bin/R, \
                            /usr/bin/python3 /opt/finance/scripts/*, \
                            /usr/bin/touch /finance/reports/*

# Finance Support - maintenance access
%finance_support ALL=(ALL) /bin/tar, \
                           /bin/gzip, \
                           /bin/mv /finance/shared/* /finance/archive/
EOF

sudo cp /tmp/10_finance /etc/sudoers.d/
sudo chmod 440 /etc/sudoers.d/10_finance
```

