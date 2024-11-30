
# AWS EC2 Master and Worker Node Setup with Ansible

This project demonstrates how to automate the setup and configuration of an Nginx web server on a Worker Node using **Ansible**. The Master Node acts as the Ansible controller, while the Worker Node hosts the Nginx web server.

---

## Table of Contents
- [Prerequisites](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Setup Steps](#setup-steps)
- [Files Included](#files-included)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Prerequisites

1. **AWS Account**: Ensure permissions to launch and manage EC2 instances.
2. **Key Pair**: Generate or use an existing SSH key pair for secure access.
3. **Security Group**: Configure to allow:
   - SSH (port 22)
   - HTTP (port 80)
   - HTTPS (port 443, optional)
4. **Software Requirements**:
   - Ansible installed on the Master Node.
   - Python3 and its dependencies on the Worker Node.

---

## Architecture Overview

1. **Master Node**:
   - Role: Ansible controller.
   - Instance Type: T2.micro or equivalent.
2. **Worker Node**:
   - Role: Hosts Nginx web server.
   - Instance Type: T2.micro or equivalent.

The communication between the nodes uses SSH, with the Master Node configuring the Worker Node using Ansible.

---

## Setup Steps

### Step 1: Launch EC2 Instances
1. Launch two EC2 instances:
   - **Master Node**: Public-facing.
   - **Worker Node**: Internal communication.
2. Apply the required security group rules.

### Step 2: Install Ansible on Master Node
1. SSH into the Master Node:
   ```bash
   ssh -i <your-key.pem> ec2-user@<Master_Public_IP>
   ```
2. Install Ansible:
   ```bash
   sudo apt update
   sudo apt install ansible -y
   ```

### Step 3: Configure SSH Access Between Nodes
1. Copy the SSH key to the Master Node:
   ```bash
   scp -i <your-key.pem> <your-key.pem> ec2-user@<Master_Public_IP>:~
   ```
2. Restrict permissions:
   ```bash
   chmod 400 <your-key.pem>
   ```

### Step 4: Prepare Ansible Inventory
1. Create an `inventory` file on the Master Node:
   ```bash
   vi inventory
   ```
2. Add the following configuration:
   ```ini
   [worker]
   <Worker_Private_IP> ansible_user=ec2-user ansible_ssh_private_key_file=~/<your-key.pem>
   ```

### Step 5: Write Ansible Playbook
1. Create the playbook file `nginx.yaml`:
   ```yaml
   ---
   - name: Configure Worker Node
     hosts: worker
     become: true
     tasks:
       - name: Update package manager cache
         yum:
           name: "*"
           state: latest

       - name: Install Nginx
         yum:
           name: nginx
           state: present

       - name: Start and enable Nginx service
         service:
           name: nginx
           state: started
           enabled: yes
   ```

### Step 6: Execute Playbook
Run the playbook:
```bash
ansible-playbook -i inventory nginx.yaml
```

### Step 7: Verify Nginx Setup
1. Check Nginx status:
   ```bash
   ansible worker -i inventory -a "systemctl status nginx"
   ```
2. Test the Nginx welcome page in a browser:
   ```
   http://<Worker_Public_IP>
   ```

---

## Files Included

The repository contains the following files:

- **inventory**: Ansible inventory file for Worker Node configuration.
- **nginx.yaml**: Ansible playbook to set up and deploy Nginx.
- **222.pem**: SSH private key (ensure proper permissions and secure handling).
- **Master and Worker Nodes.png**: Screenshot of the EC2 instances in the AWS Console.
- **Nginx Welcome Page.png**: Screenshot showing the Nginx welcome page.
- **Setup_AWS_Asible_Nginx_Commands.md**: Detailed command reference for the setup process.
- **Set Up AWS Instances Security.png**: Security group configuration screenshot.

---

## Troubleshooting

1. **Connection Issues**:
   - Verify the SSH key permissions: `chmod 400 <your-key.pem>`.
   - Ensure correct private IP in the inventory file.
   - Check security group rules for open ports.

2. **Playbook Failures**:
   - Run with verbosity: `ansible-playbook -i inventory nginx.yaml -v`.
   - Review error logs on the Worker Node.

3. **Nginx Not Accessible**:
   - Ensure port 80 is open in the Worker Node's security group.
   - Check the Nginx service status.

---

## License
This project is licensed under the MIT License.
