
# AWS EC2 Setup with Ansible to Deploy Nginx

## 1. Set Up AWS Instances
### Steps:
1. **Launch Two EC2 Instances:**
   - **Master Node:** Used for Ansible installation and management.
   - **Worker Node:** Used for configuring and testing Nginx.

2. **Configuration for Both Instances:**
   - Security Groups must allow:
     - SSH (port 22)
     - HTTP/HTTPS (ports 80/443)
   - Ensure usage of a Key Pair for secure access.

---

## 2. Install Ansible on the Master Node
### Steps:
1. SSH into the Master Node:
   ```bash
   ssh -i /path/to/your-key.pem ec2-user@<Master_Public_IP>
   ```

2. Install Ansible:
   ```bash
   sudo apt update
   sudo apt install ansible -y
   ```

---

## 3. Transfer Key to Master Node
### Steps:
1. Locate the key file on your local machine:
   ```bash
   find ~ -name "your-key.pem"
   ```

2. Transfer the key file to the Master Node:
   ```bash
   scp -i /path/to/your-key.pem /path/to/your-key.pem ec2-user@<Master_Public_IP>:~
   ```

3. Restrict permissions on the key file:
   ```bash
   ssh -i /path/to/your-key.pem ec2-user@<Master_Public_IP>
   chmod 400 ~/your-key.pem
   ```

---

## 4. Verify the Upload
### Steps:
1. Log in to the Master Node and check for the file:
   ```bash
   ssh -i 222.pem ec2-user@<Master_Public_IP>
   ls -l ~/222.pem
   ```

2. If transferring a private key:
   ```bash
   cat ~/.ssh/your.pem
   ```
   Copy the private key content and paste it into the `222.pem` file on the Master Node:
   ```bash
   cd ~/.ssh
   vi 222.pem
   chmod 600 ~/222.pem
   ```

---

## 5. Connect Master Node to Worker Node
### Steps:
1. SSH into the Worker Node using the Master Node:
   ```bash
   ssh -i ~/222.pem ec2-user@<Worker_Node_Private_IP>
   ```

2. Install necessary tools on the Worker Node:
   ```bash
   sudo yum install python3 -y
   sudo yum install python3-pip -y
   ```

3. Verify the connection:
   ```bash
   ssh -i /home/ec2-user/222.pem ec2-user@<Worker_Node_Private_IP>
   ```

4. Return to the Master Node:
   ```bash
   exit
   ```

---

## 6. Set Up Ansible Inventory File
### Steps:
1. Create an inventory file:
   ```bash
   vi ~/inventory.yaml
   ```

2. Add the following configuration:
   ```yaml
   [worker]
   <Worker_Private_IP> ansible_user=ec2-user ansible_ssh_private_key_file=~/222.pem
   ```

---

## 7. Test Ansible Connectivity
### Steps:
1. Test Ansible connectivity to the Worker Node:
   ```bash
   ansible -i ~/inventory.yaml all -m ping
   ```

---

## 8. Create and Execute the Playbook
### Steps:
1. Create a playbook for Nginx installation:
   ```bash
   vi ~/nginx.yaml
   ```

2. Add the following configuration:
   ```yaml
   ---
   - name: Configure Worker Node
     hosts: worker
     become: yes
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

3. Run the playbook:
   ```bash
   ansible-playbook -i ~/inventory.yaml ~/nginx.yaml
   ```

---

## 9. Verify Nginx Installation
### Steps:
1. Check Nginx status on the Worker Node:
   ```bash
   ansible worker -i ~/inventory.yaml -a "systemctl status nginx"
   ```

2. Test the Nginx default page using `curl`:
   ```bash
   curl http://<Worker_Node_Public_IP>
   ```

3. Alternatively, open the following in your browser:
   ```
   http://<Worker_Node_Public_IP>
   ```

---

## Troubleshooting and Notes
1. Ensure proper file permissions for private keys:
   ```bash
   chmod 400 ~/222.pem
   ```

2. Verify Ansible inventory file formatting and accuracy.
3. Confirm Security Group rules allow HTTP traffic on port 80.
4. If connectivity fails, ensure proper private key usage and host configurations.

---

## Expected Results
1. Successful playbook execution should display:
   ```
   TASK [Install Nginx] ******************************************************************
   ok: [worker]
   ```

2. Testing with `curl` or browser access should show the default Nginx welcome page.

This guide provides a complete setup process for using Ansible to configure an Nginx web server on a Worker Node managed from a Master Node in AWS EC2.
