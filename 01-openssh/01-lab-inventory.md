## Ansible Inventory: A Beginner's Lab

This lab will guide you through the basics of creating and using Ansible inventory files. You'll learn how to define hosts, group them, and apply variables.

**Scenario:** You're a system administrator managing a small network of servers. You need to automate tasks on these servers using Ansible.

---

### **Part 1: Understanding Inventory Basics**

**1. What is Ansible Inventory?**

Ansible uses an "inventory" to know which servers it should manage. This inventory is essentially a list of managed nodes (often called "hosts") that Ansible can connect to and run tasks on.

**2. Default Inventory Location:**

By default, Ansible looks for an inventory file at `/etc/ansible/hosts`. However, it's often more practical to create inventory files specific to your projects.

---

### **Part 2: Creating Your First Inventory File**

Let's create a simple inventory file to manage two web servers and a database server.

**1. Create a Project Directory:**

First, create a directory for this lab:

```bash
mkdir ansible_inventory_lab
cd ansible_inventory_lab
```

**2. Create the Inventory File:**

Now, create a file named `inventory.ini` inside your `ansible_inventory_lab` directory:

```bash
nano inventory.ini
```

**3. Add Hosts to the Inventory:**

Inside `inventory.ini`, add the following content. We'll use example IP addresses; in a real scenario, these would be your actual server IPs or hostnames.

```ini
[webservers]
web1.example.com ansible_host=192.168.1.101
web2.example.com ansible_host=192.168.1.102

[databases]
db1.example.com ansible_host=192.168.1.201

[all:vars]
ansible_user=your_ssh_username
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**Explanation:**

*   **`[webservers]` and `[databases]`**: These are "groups." Groups allow you to organize your hosts logically and target specific sets of servers.
*   **`web1.example.com`, `web2.example.com`, `db1.example.com`**: These are the "hostnames" or "aliases" you'll use to refer to your servers in Ansible playbooks.
*   **`ansible_host=192.168.1.x`**: This is a host variable that tells Ansible the actual IP address or resolvable hostname to connect to.
*   **`[all:vars]`**: This special group defines variables that apply to *all* hosts in your inventory.
    *   **`ansible_user`**: The SSH username Ansible will use to connect to your hosts. **Replace `your_ssh_username` with your actual SSH username.**
    *   **`ansible_ssh_private_key_file`**: The path to your SSH private key. Make sure this path is correct for your system.

---

### **Part 3: Testing Your Inventory**

You can use the `ansible` command with the `-i` flag to specify your inventory file and verify its contents.

**1. List All Hosts:**

To see all hosts in your inventory:

```bash
ansible all -i inventory.ini --list-hosts
```

You should see output similar to this:

```
  hosts (3):
    web1.example.com
    web2.example.com
    db1.example.com
```

**2. List Hosts in a Specific Group:**

To see hosts belonging to the `webservers` group:

```bash
ansible webservers -i inventory.ini --list-hosts
```

Output:

```
  hosts (2):
    web1.example.com
    web2.example.com
```

**3. Ping Your Hosts (Dry Run):**

The `ping` module is a simple way to test connectivity and authentication. Ansible's `ping` module actually just checks if the remote host can respond.

```bash
ansible all -i inventory.ini -m ping
```

**Expected Output (if successful):**

```
web1.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
db1.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
web2.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

**Troubleshooting `UNREACHABLE` or `FAILED`:**

*   **SSH Key:** Double-check `ansible_ssh_private_key_file` path and permissions (`chmod 400 ~/.ssh/id_rsa`).
*   **SSH User:** Ensure `ansible_user` is correct and has SSH access to the target machines.
*   **Firewall:** Make sure port 22 (SSH) is open on your target hosts.
*   **IP Address:** Verify the `ansible_host` IP addresses are correct and reachable from your control machine.
*   **Python:** Ensure Python is installed on your target hosts (Ansible needs it to run modules).

---

### **Part 4: Adding Group Variables**

Variables can be applied at the group level, making it easy to manage configurations specific to a group of servers.

**1. Update `inventory.ini`:**

Let's say our web servers use Nginx and our database server uses PostgreSQL. We can define variables for these. Modify your `inventory.ini` to include these:

```ini
[webservers]
web1.example.com ansible_host=192.168.1.101
web2.example.com ansible_host=192.168.1.102

[databases]
db1.example.com ansible_host=192.168.1.201

[webservers:vars]
http_port=80
backend_server="appserver.example.com"
web_service="nginx"

[databases:vars]
db_port=5432
db_engine="postgresql"

[all:vars]
ansible_user=your_ssh_username
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**2. Explore Variables with `debug`:**

You can use a simple playbook to see how these variables are applied.

Create a file named `debug_vars.yml`:

```yaml
---
- name: Debugging inventory variables
  hosts: all
  gather_facts: false # No need to gather facts for this example
  tasks:
    - name: Show host and group variables
      debug:
        msg: "Hostname: {{ inventory_hostname }}, HTTP Port: {{ http_port | default('N/A') }}, DB Port: {{ db_port | default('N/A') }}, Web Service: {{ web_service | default('N/A') }}, DB Engine: {{ db_engine | default('N/A') }}"
```

**Explanation:**

*   `inventory_hostname`: A built-in Ansible variable that refers to the current host's name as defined in inventory.
*   `{{ variable_name | default('N/A') }}`: This Jinja2 filter displays the variable's value if it exists, otherwise it shows "N/A" to prevent errors if a variable isn't defined for a specific host.

**3. Run the Playbook:**

```bash
ansible-playbook -i inventory.ini debug_vars.yml
```

**Expected Output:**

You'll see output for each host, showing the variables specific to its group and any `all:vars`.

```
PLAY [Debugging inventory variables] *******************************************

TASK [Show host and group variables] *******************************************
ok: [web1.example.com] => {
    "msg": "Hostname: web1.example.com, HTTP Port: 80, DB Port: N/A, Web Service: nginx, DB Engine: N/A"
}
ok: [web2.example.com] => {
    "msg": "Hostname: web2.example.com, HTTP Port: 80, DB Port: N/A, Web Service: nginx, DB Engine: N/A"
}
ok: [db1.example.com] => {
    "msg": "Hostname: db1.example.com, HTTP Port: N/A, DB Port: 5432, Web Service: N/A, DB Engine: postgresql"
}

PLAY RECAP *********************************************************************
db1.example.com            : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web1.example.com           : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2.example.com           : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Notice how `http_port` and `web_service` are set for web servers, and `db_port` and `db_engine` are set for the database server, but not vice-versa.

---

### **Part 5: Dynamic Inventory (Conceptual)**

While this lab focuses on static inventory, it's important to know about dynamic inventory.

**What is Dynamic Inventory?**

Dynamic inventory uses scripts to pull host information from cloud providers (AWS, Azure, GCP), virtualization platforms (VMware), or configuration management databases (CMDBs). This is crucial in environments where servers are frequently spun up and down.

**When to use it:**

*   Cloud environments (VMs come and go).
*   Highly dynamic infrastructure.
*   When managing hundreds or thousands of hosts.

**Example (Conceptual):**

Instead of manually typing IPs, a script would query AWS and generate an inventory of all running EC2 instances, automatically grouping them by tags, region, etc.

---

### **Part 6: Bonus - Host Variables**

You can also define variables for individual hosts. These override group variables.

**1. Update `inventory.ini`:**

Let's say `web1.example.com` needs a different `http_port` for some reason.

```ini
[webservers]
web1.example.com ansible_host=192.168.1.101 http_port=8080 # Host-specific variable
web2.example.com ansible_host=192.168.1.102

[databases]
db1.example.com ansible_host=192.168.1.201

[webservers:vars]
http_port=80 # Group variable
backend_server="appserver.example.com"
web_service="nginx"

[databases:vars]
db_port=5432
db_engine="postgresql"

[all:vars]
ansible_user=your_ssh_username
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**2. Rerun the Debug Playbook:**

```bash
ansible-playbook -i inventory.ini debug_vars.yml
```

**Expected Change in Output:**

Now, `web1.example.com` will show `HTTP Port: 8080`, while `web2.example.com` will still show `HTTP Port: 80`. Host variables take precedence!

```
PLAY [Debugging inventory variables] *******************************************

TASK [Show host and group variables] *******************************************
ok: [web1.example.com] => {
    "msg": "Hostname: web1.example.com, HTTP Port: 8080, DB Port: N/A, Web Service: nginx, DB Engine: N/A"
}
ok: [web2.example.com] => {
    "msg": "Hostname: web2.example.com, HTTP Port: 80, DB Port: N/A, Web Service: nginx, DB Engine: N/A"
}
ok: [db1.example.com] => {
    "msg": "Hostname: db1.example.com, HTTP Port: N/A, DB Port: 5432, Web Service: N/A, DB Engine: postgresql"
}

PLAY RECAP *********************************************************************
db1.example.com            : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web1.example.com           : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2.example.com           : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
Here's a visual representation of how different parts of an Ansible inventory file logically organize hosts and variables.

```mermaid
graph TD
    A[Inventory File] --> B[Host Group: webservers]
    A --> C[Host Group: databases]
    A --> D[All Hosts Variables]

    B --> B1[web1.example.com]
    B --> B2[web2.example.com]
    C --> C1[db1.example.com]

    B1 --> B1a(ansible_host=192.168.1.101)
    B1 --> B1b(http_port=8080 - Host Variable)

    B2 --> B2a(ansible_host=192.168.1.102)

    C1 --> C1a(ansible_host=192.168.1.201)

    B --> B_vars[webservers:vars]
    B_vars --> B_vars1(http_port=80 - Group Variable)
    B_vars --> B_vars2(backend_server="appserver.example.com")
    B_vars --> B_vars3(web_service="nginx")

    C --> C_vars[databases:vars]
    C_vars --> C_vars1(db_port=5432)
    C_vars --> C_vars2(db_engine="postgresql")

    D --> D1(ansible_user=your_ssh_username)
    D --> D2(ansible_ssh_private_key_file=~/.ssh/id_rsa)

    style B1b fill:#f9f,stroke:#333,stroke-width:2px
    style B_vars1 fill:#cfc,stroke:#333,stroke-width:2px

    subgraph Variable Precedence
        E[Host Variable] --> F[Group Variable]
        F --> G[All Hosts Variable]
        E--Overrides-->F
        F--Overrides-->G
    end
```
This diagram illustrates the hierarchical structure of an Ansible inventory. Individual hosts can be part of one or more groups, and variables can be defined at the 'all' level, group level, or host level. Variables defined at a more specific level (like host variables) will override those defined at a broader level (like group or all variables).

---

### **Conclusion:**

You've successfully created a basic Ansible inventory, defined hosts and groups, applied both `all` and group-specific variables, and understood variable precedence. This is a fundamental step in managing your infrastructure with Ansible!

**Next Steps:**

*   Explore more complex inventory structures, including nested groups.
*   Learn about `host_vars/` and `group_vars/` directories for better organization of variables.
*   Research dynamic inventory plugins for your specific cloud provider or virtualization platform.
