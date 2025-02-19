
# Ansible Nginx Deployment with Terraform

This repository contains the necessary configurations to deploy two EC2 instances, install and configure Nginx using Ansible, and set up a static inventory for managed nodes.

## **Setup Instructions**

### **1. Generate SSH Key Pair**
Run the following command to create a new SSH key pair named `aws`:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws -N ""
```

This will generate:

- Private key: `~/.ssh/aws`
- Public key: `~/.ssh/aws.pub`

### **2. Import the SSH Public Key to AWS**
Use the provided script to import your public key:

```bash
./import_lab_key.sh ~/.ssh/aws.pub
```

### **3. Deploy EC2 Instances using Terraform**
Initialize and apply the Terraform configuration:

```bash
terraform init
terraform apply -auto-approve
```

After completion, Terraform will output the public IP addresses and DNS names of the instances.

### **4. Configure Ansible Inventory**
Update `ansible/inventory/hosts.yml` with the instance details:

```yaml
all:
  children:
    web:
      hosts:
        server-one:
          ansible_host: <public-ip-or-dns-of-first-instance>
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/aws
        server-two:
          ansible_host: <public-ip-or-dns-of-second-instance>
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/aws
```

### **5. Modify the Ansible Playbook**
Update `ansible/playbook.yml` to configure Nginx:

```yaml
- name: Configure web servers
  hosts: web
  become: yes
  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Create directory structure for web documents
      ansible.builtin.file:
        path: /var/www/html
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'

    - name: Copy nginx conf file to server
      ansible.builtin.copy:
        src: files/nginx.conf
        dest: /etc/nginx/sites-available/default
        owner: root
        group: root
        mode: '0644'

    - name: Create symbolic link to enable nginx site
      ansible.builtin.file:
        src: /etc/nginx/sites-available/default
        dest: /etc/nginx/sites-enabled/default
        state: link

    - name: Generate index.html from template
      ansible.builtin.template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Reload and enable nginx service
      ansible.builtin.service:
        name: nginx
        state: reloaded
        enabled: yes
```

### **6. Run the Ansible Playbook**
Execute the playbook to install and configure Nginx:

```bash
ansible-playbook -i ansible/inventory/hosts.yml playbook.yml
```

### **7. Verify the Deployment**
Visit `http://<public-ip-or-dns>` in your browser. If successful, you should see the deployed Nginx webpage.

### **8. Screenshot of Rendered HTML**
Below is a screenshot of the successfully deployed webpage:
![image](https://github.com/user-attachments/assets/3a743811-3a22-4d59-8c1d-e33d305a90d9)
