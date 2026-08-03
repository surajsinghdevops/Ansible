
# Ansible Master and Slave Setup on Amazon Linux 2023

## Overview

This project demonstrates how to configure an **Ansible Control Node (Master)** and a **Managed Node (Slave)** on Amazon EC2 instances running Amazon Linux.

After completing this setup, the master server will be able to manage the slave server using SSH without requiring a password.

---

# Architecture

```
                   +----------------------+
                   |  Ansible Master      |
                   |  Amazon Linux 2023   |
                   |  Control Node        |
                   +----------+-----------+
                              |
                              | SSH (Key Authentication)
                              |
                   +----------v-----------+
                   |  Ansible Slave       |
                   |  Amazon Linux 2023   |
                   |  Managed Node        |
                   +----------------------+
```

---

# Prerequisites

- AWS Account
- Two EC2 Instances (Amazon Linux 2023)
- Same VPC
- Same Security Group (or allow SSH between them)
- Key Pair for EC2 login
- Internet Access

---

# EC2 Configuration

## Master Server

- Instance Type : t3.micro
- OS : Amazon Linux 2023
- Name : ansible-master

---

## Slave Server

- Instance Type : t3.micro
- OS : Amazon Linux 2023
- Name : ansible-slave

---

# Security Group

Allow the following inbound rule.

| Type | Port | Source |
|-------|------|---------|
| SSH | 22 | Your IP or Security Group |

---

# Step 1 : Update Both Servers

```bash
sudo yum update -y
```

---

# Step 2 : Install Ansible on Master

```bash
sudo yum install ansible -y
```

Verify installation

```bash
ansible --version
```

---

# Step 3 : Enable Root Login on Slave

Edit SSH configuration.

```bash
sudo vi /etc/ssh/sshd_config
```

Modify or uncomment the following parameters.

```text
PermitRootLogin yes

PasswordAuthentication yes
```

Restart SSH service.

```bash
sudo systemctl restart sshd
```

Enable SSH service.

```bash
sudo systemctl enable sshd
```

---

# Step 4 : Set Root Password on Slave

Switch to root.

```bash
sudo su -
```

Set password.

```bash
passwd
```

Example password

```
1
```

*(For production environments, always use a strong password.)*

---

# Step 5 : Generate SSH Key on Master

Login to master server.

Generate SSH key.

```bash
ssh-keygen
```

Press Enter for all prompts.

This creates

```
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

---

# Step 6 : Copy SSH Key to Slave

Install ssh-copy-id if required.

```bash
sudo yum install openssh-clients -y
```

Copy the public key.

```bash
ssh-copy-id root@<Slave-Private-IP>
```

Example

```bash
ssh-copy-id root@172.31.10.25
```

Enter the root password when prompted.

Verify passwordless login.

```bash
ssh root@172.31.10.25
```

You should login without entering a password.

---

# Step 7 : Configure Inventory

Navigate to the Ansible directory.

```bash
cd /etc/ansible
```

Edit the hosts file.

```bash
sudo vi hosts
```

Example

```ini
[web]

172.31.10.25
```

or

```ini
[web]
ansible-slave ansible_host=172.31.10.25 ansible_user=root
```

---

# Step 8 : Verify Inventory

```bash
ansible-inventory --list
```

---

# Step 9 : Test Connectivity

Ping the managed node.

```bash
ansible web -m ping
```

Expected Output

```json
172.31.10.25 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# Useful Ansible Commands

List hosts

```bash
ansible all --list-hosts
```

Check connectivity

```bash
ansible all -m ping
```

Gather system information

```bash
ansible all -m setup
```

Check uptime

```bash
ansible all -a "uptime"
```

Check hostname

```bash
ansible all -a "hostname"
```

Check disk usage

```bash
ansible all -a "df -h"
```

Check memory

```bash
ansible all -a "free -m"
```

Install Apache

```bash
ansible web -m yum -a "name=httpd state=present"
```

Start Apache

```bash
ansible web -m service -a "name=httpd state=started enabled=yes"
```

---

# Directory Structure

```
.
├── README.md
└── inventory
```

---

# Troubleshooting

### Permission denied (publickey)

Verify:

- SSH key exists
- Public key copied correctly
- Root login enabled
- Correct private IP

---

### Host unreachable

Check

- Security Group
- SSH Service
- Private IP
- Both instances are in the same VPC

---

### Ping Failed

Verify inventory

```bash
ansible-inventory --list
```

Check SSH manually

```bash
ssh root@<private-ip>
```

---

# Best Practices

- Disable root login in production.
- Use SSH key authentication only.
- Create a dedicated Ansible user instead of using root.
- Protect private SSH keys.
- Organize hosts using inventory groups.
- Store playbooks in a Git repository.
- Use Ansible Vault for sensitive information.

---

# Conclusion

You have successfully configured:

- Amazon Linux Control Node (Master)
- Amazon Linux Managed Node (Slave)
- Passwordless SSH Authentication
- Ansible Inventory
- Connectivity Verification using `ansible ping`

The environment is now ready for running Ansible playbooks and automating infrastructure management.

---

## Author

**Suraj Singh R**

DevOps | AWS | Linux | Docker | Kubernetes | Terraform | Ansible
