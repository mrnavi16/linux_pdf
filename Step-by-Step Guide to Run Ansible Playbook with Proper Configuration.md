# Step-by-Step Guide to Run Ansible Playbook with Proper Configuration

This document provides all the steps and commands required to configure and run an Ansible playbook securely and successfully. Follow the steps carefully to resolve issues and achieve the desired results.

---

## 1. **Set Up SSH for the Target Machine**

1. **Generate an SSH Key Pair (If Not Already Done):**
   ```bash
   ssh-keygen -t rsa -b 4096
   ```

2. **Copy the Public Key to the Target Machine:**
   ```bash
   ssh-copy-id opstree@10.10.2.55
   ```

3. **Verify SSH Connectivity:**
   ```bash
   ssh opstree@10.10.2.55
   ```
   If successful, proceed to the next steps.

---

## 2. **Prepare the Inventory File**

1. **Create the Inventory File `inventory.ini`:**
   ```ini
   [ubuntu_servers]
   10.10.2.55 ansible_user=opstree ansible_ssh_private_key_file=~/.ssh/id_rsa
   ```

2. **Test the Inventory File:**
   ```bash
   ansible-inventory -i inventory.ini --list
   ```
   Ensure the target host (`10.10.2.55`) appears correctly in the output.

3. **Test Connectivity to the Host:**
   ```bash
   ansible -i inventory.ini ubuntu_servers -m ping
   ```
   Expected output:
   ```json
   10.10.2.55 | SUCCESS => {
       "changed": false,
       "ping": "pong"
   }
   ```

---

## 3. **Configure the Playbook**

1. **Create the Ansible Playbook `secure_ubuntu.yml`:**
   ```yaml
   ---
   - name: Secure Ubuntu system
     hosts: ubuntu_servers
     become: yes
     tasks:
       - name: Install security updates
         apt:
           upgrade: dist
           update_cache: yes
           cache_valid_time: 3600

       - name: Disable root login via SSH
         lineinfile:
           path: /etc/ssh/sshd_config
           regexp: '^PermitRootLogin'
           line: 'PermitRootLogin no'

       - name: Ensure UFW is installed and enabled
         ufw:
           state: enabled
           policy: deny
   ```

2. **Check Playbook Syntax:**
   ```bash
   ansible-playbook --syntax-check secure_ubuntu.yml
   ```

---

## 4. **Run the Playbook with Sudo Password**

1. **Option 1: Use `--ask-become-pass` to Provide Password Interactively:**
   ```bash
   ansible-playbook -i inventory.ini secure_ubuntu.yml --ask-become-pass
   ```

2. **Option 2: Use Ansible Vault for Secure Password Storage:**

   - **Create a Vault File for the Sudo Password:**
     ```bash
     ansible-vault create vault.yml
     ```

   - **Add the Password to the Vault File:**
     ```yaml
     ansible_become_password: your_sudo_password
     ```

   - **Update the Playbook to Use the Vault File:**
     ```yaml
     ---
     - name: Secure Ubuntu system
       hosts: ubuntu_servers
       become: yes
       vars_files:
         - vault.yml
       tasks:
         - name: Install security updates
           apt:
             upgrade: dist
             update_cache: yes
             cache_valid_time: 3600
     ```

   - **Run the Playbook with the Vault Password:**
     ```bash
     ansible-playbook -i inventory.ini secure_ubuntu.yml --ask-vault-pass
     ```

---

## 5. **Package Management Hardening (CIS Benchmark)**

The following steps outline how to implement CIS-compliant package management configurations on Ubuntu using Ansible:

1. **Ensure Only Approved Repositories Are Enabled**
   ```yaml
   - name: Ensure only approved repositories are enabled
     hosts: ubuntu_servers
     become: yes
     tasks:
       - name: Check repository files
         command: grep -E '^[^#]' /etc/apt/sources.list /etc/apt/sources.list.d/*
         register: repo_files

       - name: Display repository configuration
         debug:
           var: repo_files.stdout_lines

       - name: Remove unapproved repositories (if any are found)
         lineinfile:
           path: "{{ item }}"
           regexp: 'unapproved_repo_url'
           state: absent
         with_items: "{{ repo_files.stdout_lines }}"
   ```

2. **Verify GPG Keys for Repositories**
   ```yaml
   - name: Verify GPG keys for repositories
     hosts: ubuntu_servers
     become: yes
     tasks:
       - name: List all GPG keys
         command: apt-key list
         register: gpg_keys

       - name: Display GPG keys
         debug:
           var: gpg_keys.stdout_lines

       - name: Remove unauthorized GPG keys
         command: apt-key del {{ item }}
         with_items:
           - "unauthorized_key_1"
           - "unauthorized_key_2"
   ```

3. **Automate Package Updates**
   ```yaml
   - name: Automate package updates
     hosts: ubuntu_servers
     become: yes
     tasks:
       - name: Install unattended-upgrades package
         apt:
           name: unattended-upgrades
           state: present

       - name: Enable automatic updates
         lineinfile:
           path: /etc/apt/apt.conf.d/20auto-upgrades
           regexp: '^\s*"\d"\s*;'
           line: 'APT::Periodic::Update-Package-Lists "1";'

       - name: Enable unattended-upgrades
         lineinfile:
           path: /etc/apt/apt.conf.d/20auto-upgrades
           regexp: '^\s*"\d"\s*;'
           line: 'APT::Periodic::Unattended-Upgrade "1";'
   ```

4. **Prevent the Installation of Specific Packages**
   ```yaml
   - name: Prevent installation of specific packages
     hosts: ubuntu_servers
     become: yes
     tasks:
       - name: Add blacklisted packages to dpkg
         lineinfile:
           path: /etc/apt/apt.conf.d/50blacklist
           line: 'Package: {{ item }}\nPin: release *\nPin-Priority: -1'
         with_items:
           - telnet
           - rsh
           - rlogin
           - talk
   ```

5. **Run the Package Management Playbook**
   Save the above tasks to a playbook file, e.g., `package_management.yml`, and run it:
   ```bash
   ansible-playbook -i inventory.ini package_management.yml --ask-become-pass
   ```

---

## 6. **Verify the Results**

1. **Check Repositories**:
   ```bash
   ssh opstree@10.10.2.55 "cat /etc/apt/sources.list"
   ```

2. **Verify GPG Keys**:
   ```bash
   ssh opstree@10.10.2.55 "apt-key list"
   ```

3. **Check Automatic Updates**:
   ```bash
   ssh opstree@10.10.2.55 "cat /etc/apt/apt.conf.d/20auto-upgrades"
   ```

4. **Confirm Blacklisted Packages**:
   ```bash
   ssh opstree@10.10.2.55 "cat /etc/apt/apt.conf.d/50blacklist"
   ```

---

## 7. **Debugging Common Issues**

1. **Test Connectivity with Verbose Output:**
   ```bash
   ansible -i inventory.ini ubuntu_servers -m ping -vvv
   ```

2. **Check Playbook Execution Logs:**
   ```bash
   ansible-playbook -i inventory.ini secure_ubuntu.yml -vvv
   ```

3. **Verify SSH Server Configuration on the Target Host:**
   ```bash
   ssh opstree@10.10.2.55 "sudo cat /etc/ssh/sshd_config"
   ```

4. **Restart SSH Service After Changes:**
   ```bash
   ssh opstree@10.10.2.55 "sudo systemctl restart sshd"
   ```

---

## 8. **Complete Command List in Order**

1. Generate SSH Key Pair:
   ```bash
   ssh-keygen -t rsa -b 4096
   ```

2. Copy SSH Public Key to Target Machine:
   ```bash
   ssh-copy-id opstree@10.10.2.55
   ```

3. Verify SSH Access:
   ```bash
   ssh opstree@10.10.2.55
   ```

4. Create `inventory.ini` File:
   ```ini
   [ubuntu_servers]
   10.10.2.55 ansible_user=opstree ansible_ssh_private_key_file=~/.ssh/id_rsa
   ```

5. Test Inventory File:
   ```bash
   ansible-inventory -i inventory.ini --list
   ```

6. Test Ping to Target Host:
   ```bash
   ansible -i inventory.ini ubuntu_servers -m ping
   ```

7. Create `secure_ubuntu.yml` Playbook:
   ```yaml
   ---
   - name: Secure Ubuntu system
     hosts: ubuntu_servers
     become: yes
     tasks:
       - name: Install security updates
         apt:
           upgrade: dist
           update_cache: yes
           cache_valid_time: 3600

       - name: Disable root login via SSH
         lineinfile:
           path: /etc/ssh/sshd_config
           regexp: '^PermitRootLogin'
           line: 'PermitRootLogin no'

       - name: Ensure UFW is installed and enabled
         ufw:
           state: enabled
           policy: deny
   ```

8. Check Playbook Syntax:
   ```bash
   ansible-playbook --syntax-check secure_ubuntu.yml
   ```

9. Run Playbook with Sudo Password:
   ```bash
   ansible-playbook -i inventory.ini secure_ubuntu.yml --ask-become-pass
   ```

10. Create and Use Ansible Vault (Optional):
    - Create Vault File:
      ```bash
      ansible-vault create vault.yml
      ```
    - Add Password to Vault File:
      ```yaml
      ansible_become_password: your_sudo_password
      ```
    - Run Playbook with Vault Password:
      ```bash
      ansible-playbook -i inventory.ini secure_ubuntu.yml --ask-vault-pass
      ```

11. Run Package Management Playbook:
    ```bash
    ansible-playbook -i inventory.ini package_management.yml --ask-become-pass
    ```

12. Verify Results:
    - Check Repositories:
      ```bash
      ssh opstree@10.10.2.55 "cat /etc/apt/sources.list"
      ```
    - Check Automatic Updates:
      ```bash
      ssh opstree@10.10.2.55 "cat /etc/apt/apt.conf.d/20auto-upgrades"
      ```
    - Confirm Blacklisted Packages:
      ```bash
      ssh opstree@10.10.2.55 "cat /etc/apt/apt.conf.d/50blacklist"
      ```

---

By following this sequence, you can set up and execute Ansible playbooks on your target Ubuntu servers efficiently.

