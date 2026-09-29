# 15 - Configuration Management with Ansible

## 1 - Introduction to Ansible

Ansible is an open-source automation tool that simplifies configuration management, application deployment, and task automation. It uses a simple, human-readable language called YAML to define automation tasks in playbooks. Ansible operates in an agentless manner, meaning it does not require any software to be installed on the target machines. Instead, it uses SSH for communication, making it easy to manage a large number of servers.

Ansible uses modules to perform specific tasks, such as installing packages, managing services, and configuring files. These modules can be used in playbooks to automate complex workflows. Ansible also supports inventory management, allowing users to define groups of hosts and apply configurations to them.

An Ansible module is a reusable, standalone script that performs a specific task on a target system. Modules can manage system resources, such as packages, services, and files, or execute commands. They are the building blocks of Ansible playbooks, allowing users to automate complex workflows efficiently.

An Ansible task is a single unit of work that is executed on a target system. Tasks are defined within playbooks and typically use modules to perform actions, such as installing a package, starting a service, or copying a file. Each task has a name, which helps in identifying it in the playbook output, and can include various parameters to control its behavior.


An Ansible host refers to a target machine or server that Ansible manages and automates tasks on

Ansible playbook is a YAML file that contains a series of tasks to be executed on one or more hosts. Playbooks are the primary way to define automation workflows in Ansible, allowing users to describe the desired state of their systems and applications. Each playbook consists of one or more plays, which define the target hosts and the tasks to be performed on them.

The host hold a reference to the target machine or server that Ansible manages and automates tasks on. It can be defined in the inventory file, which lists all the hosts and their associated groups. Ansible uses this inventory to determine which hosts to target when executing playbooks.

```yaml
# Example of an Ansible playbook
- name: Install and start Apache web server
  hosts: webservers
  become: yes
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
    - name: Start Apache service
        service: 
          name: apache2
          state: started
```

Sample Inventory File:

```ini
# Example of an Ansible inventory file
[webservers]
webserver1 ansible_host=192.168.1.10
webserver2 ansible_host=

[databases]
dbserver1 ansible_host=192.168.1.20
dbserver2 ansible_host=192.168.1.21
```

Ansible Tower is a web-based interface for managing Ansible automation. It provides a centralized platform for managing playbooks, inventories, and credentials, as well as scheduling and monitoring automation tasks. Ansible Tower also offers role-based access control, allowing organizations to manage permissions and access to automation resources effectively.

Alternative Ansible tools include puppet, Chef, and SaltStack. These tools also provide configuration management and automation capabilities, but they differ in their architecture, language, and approach to automation. Ansible is known for its simplicity and ease of use, making it a popular choice for many organizations.

Ansible vs puppet & chef table

| Feature          | Ansible                  | Puppet                   | Chef                     |
|------------------|--------------------------|--------------------------|--------------------------|
| Language         | YAML                     | Puppet DSL               | Ruby DSL                 |
| Architecture     | Agentless                | Agent-based              | Agent-based              |
| Ease of Use      | Easy                     | Moderate                 | Moderate                 |
| Configuration    | Playbooks                | Manifests                | Recipes                  |
| Community        | Large and active         | Large and active         | Large and active         |
| Use Case         | Configuration management | Configuration management | Configuration management |

## 2 - Install Ansible

Ansible can either be installed on a control node (a machine from which you run Ansible commands) or on a local machine. The installation process may vary depending on the operating system being used. Below are the steps to install Ansible on Mac:

```bash
brew install ansible
```

Ansible is written in Python, so it can also be installed using pip, the Python package manager. To install Ansible using pip, run the following command:

```bash
pip install ansible
```

## 3 - Setup Managed Server to Configure with Ansible

Create 2 droplets in digital ocean to configure with ansible. 

Ansible uses SSH to connect to the managed servers, so you need to ensure that SSH access is set up correctly. You can use SSH keys for authentication, which is more secure than using passwords.

Ansible requires Python to be installed on the managed servers. Most Linux distributions come with Python pre-installed, but you may need to install it manually on some systems.

## 4 - Ansible Inventory and Ansible ad-hoc commands

To connect ansible to the managed servers, you need to create an inventory file that lists the hosts and their associated groups. The inventory file can be in INI or YAML format. the default location for the inventory file is /etc/ansible/hosts, but you can also specify a custom location using the -i option when running ansible commands.

Ansible needs to authenticate with either username and password or SSH key to connect to the managed servers. You can specify the authentication method in the inventory file or use command-line options when running ansible commands.

Example of an Ansible inventory file in INI format, create "hosts" file with the following content:

```ini
159.89.167.194 ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_user=root
64.227.169.232 ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_user=root
```

Ansible ad-hoc commands are one-time commands that can be executed on managed servers without creating a playbook. Ad-hoc commands are useful for performing quick tasks, such as checking the status of a service or installing a package. To run an ad-hoc command, use the ansible command followed by the target hosts and the module to be executed.

```bash
# pattern: target hosts or groups defined in the inventory file (e.g., webservers, databases, all)
# module: Ansible module to be executed (e.g., ping, shell, command, etc.)
# arguments: optional arguments or module options to customize the command
ansible [pattern] -m [module] -a "[arguments or module options]"

ansible all -i hosts -m ping
``` 

In hosts file you can group the servers.

- You can put each host in more than 1 group
- you can create groups that track
  - where - a datacenter or region (e.g., us-east, eu-west)
  - what - a specific role or function (e.g., webservers, databases)
  - when - a specific environment (e.g., production, staging, development)

```ini
[droplet]
159.89.167.194 ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_user=root
64.227.169.232 ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_user=root

[webservers]

[databases]
```

To target the droplet group, you can use the following command:

```bash
# target all hosts
ansible all -i hosts -m ping

# target all hosts in the droplet group and execute the ping module
ansible droplet -i hosts -m ping

# target 1 server in the droplet group and execute the ping module
ansible droplet -i hosts -m ping -l 159.89.167.194
# or
ansible 159.89.167.194 -i hosts -m ping
```

Instead of repeating the variables in the inventory file, you can create a group_vars entry or directory and create a YAML file for each group. For example, you can create a group_vars/droplet.yml file with the following content:

```ini
[droplet]
159.89.167.194
64.227.169.232

[droplet:vars]
ansible_ssh_private_key_file: ~/.ssh/id_ed25519
ansible_user: root
```

## 5 - Configure AWS EC2 server with Ansible

Create 2 EC2 instances in AWS to configure with ansible. Download the private key from AWS and update the file permission to 400. 

```bash
chmod 400 <private-key-file>.pem
```

In ansible inventory file, you can specify the path to the private key file and the username for the EC2 instances. For example, you can create a hosts file with the following content:

```ini
[droplet]
159.89.167.194
64.227.169.232

[droplet:vars]
ansible_ssh_private_key_file: ~/.ssh/id_ed25519
ansible_user: root

[ec2]
ec2-3-120-45-67.compute-1.amazonaws.com
ec2-3-120-45-68.compute-1.amazonaws.com

[ec2:vars]
ansible_ssh_private_key_file=~/.ssh/my-aws-key.pem
ansible_user=ec2-user
# suppress python interpreter warning by specifying the python interpreter path
ansible_python_interpreter=/usr/bin/python3.12
```

## 6 - Managing Host Key Checking and SSH keys

Host key checking is enabled bu default in Ansible to ensure the authenticity of the target hosts, it guards against server spoofing and man-in-the-middle attacks. When connecting to a new host for the first time, Ansible will prompt you to confirm the host's fingerprint. If you want to disable host key checking, you can set the `ANSIBLE_HOST_KEY_CHECKING` environment variable to `False` or add the following line to your ansible.cfg file:

```ini
[defaults]
host_key_checking = False
```

If you don't want to disable ssh host key checking:

SSH host checking requires both the local machine & the target server to allow each other to connect. The local machine needs to include the target server in its known_hosts file (~/.ssh/known_hosts), and the target server needs to include the local machine public key in its authorized_keys file (~/.ssh/authorized_keys).

To add the target server to the known_hosts file, you can use the following command:

```bash
ssh-keyscan -H <target-server-ip> >> ~/.ssh/known_hosts

# check all know hosts
cat ~/.ssh/known_hosts
```

When you create a droplet with public ssh key, the droplet will automatically add the public key to its authorized_keys file. If you want to add a new public key to the target server, you can use the following command:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub <username>@<target-server-ip>

# check all authorized keys on the target server
ssh <username>@<target-server-ip> "cat ~/.ssh/authorized_keys"
```

For ephemeral infrastructure we can disable ssh host key checking in ansible config file (/etc/ansible/ansible.cfg) or in (~/.ansible.cfg) or in the inventory file by adding the following line:

```ini
[defaults]
host_key_checking = False
```

For configuration ansible will process the following list and use the first file found & ignore the rest:

- ANSIBLE_CONFIG (an environment variable)
- ansible.cfg (in the current directory)
- ~/.ansible.cfg (in the home directory)
- /etc/ansible/ansible.cfg (in the /etc/ansible directory)

## 7 - Introduction to Playbooks

Create "hosts" file with the following content:

```ini
[webserver]
159.89.167.194
64.227.169.232

[webserver:vars]
ansible_ssh_private_key_file: ~/.ssh/id_ed25519
ansible_user: root
ansible_python_interpreter=/usr/bin/python3.12
```

Create ansible config file "ansible.cfg" with the following content:

```ini
[defaults]
inventory = hosts
host_key_checking = False
```

Create "my-playbook.yml" file with the following content:

```yaml
---
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install nginx server
      apt:
        name: nginx
        state: latest

    - name: Start nginx service
      service:
        name: nginx
        state: started
```

Run the playbook with the following command:

```bash
ansible-playbook my-playbook.yml
# or if hosts is not specified in ansible.cfg file
ansible-playbook -i hosts my-playbook.yml
```

Once you run the playbook, you'll see the "Gathering Facts" module which gets executed automatically to gather some variables about the target hosts that can be used in playbooks. After that, the tasks defined in the playbook will be executed on the target hosts. You can check the status of the nginx service on the target hosts by running the following command:

```bash
ansible webserver -m service -a "name=nginx state=started"
```

SSH to the server & check the status of the nginx service:

```bash
ssh root@ip-address
systemctl status nginx
# or
ps aux | grep nginx
```

We can specify the version to install in the playbook by adding the version number to the package name. 

```yaml
---
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install nginx server
      apt:
        name: nginx=1.24.0-1ubuntu1 # or regex 1.24.*
        state: present # state preset 

    - name: Start nginx service
      service:
        name: nginx
        state: started
```

To stop & uninstall nginx service, you can use the following playbook:

```yaml
---
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Uninstall nginx server
      apt:
        name: nginx=1.24.0-1ubuntu1 # or regex 1.24.*
        state: absent # state absent

    - name: Stop nginx service
      service:
        name: nginx
        state: stopped
```

Ansible is idempotent, meaning that running the same playbook multiple times will not change the state of the system if it is already in the desired state. This ensures that playbooks can be safely re-run without causing unintended side effects.

## 8 - Modules & Collections in Ansible

What is ansible collection? 

Ansible collections are a distribution format for Ansible content that can include playbooks, roles, modules, and plugins. Collections allow users to package and distribute their Ansible content in a standardized way, making it easier to share and reuse automation code. Collections can be installed from Ansible Galaxy or other sources, and they can be used in playbooks just like any other Ansible content.

Check Ansible docs for modules https://docs.ansible.com/projects/ansible/latest/collections/index_module.html & collections: https://docs.ansible.com/ansible/latest/collections/index.html

Ansible plugin is a piece of code that extends the functionality of Ansible. Plugins can be used to customize the behavior of Ansible, such as adding new modules, modifying existing modules, or changing the way Ansible interacts with target systems. There are several types of plugins in Ansible, including action plugins, callback plugins, connection plugins, and inventory plugins.

Ansible Galaxy is a community-driven platform for sharing and discovering Ansible content, including roles, collections, and playbooks. It provides a centralized repository where users can find and download pre-built automation code, as well as contribute their own content to the community. the url for ansible galaxy is https://galaxy.ansible.com/

To install a collection from Ansible Galaxy, you can use the ansible-galaxy command-line tool. For example, to install the community.general collection, run the following command:

```bash
ansible-galaxy collection install community.general
```

To list the local collections created when installing ansible, you can use the following command:

```bash
ansible-galaxy collection list
```

You can create your own collection by using the ansible-galaxy command-line tool. For example, to create a new collection named my_collection, run the following command:

```bash
ansible-galaxy collection init my_collection
```

Collections have a specific directory structure

```text
collection/
├── docs/
├── galaxy.yml
├── meta/
│   └── runtime.yml
├── plugins/
│   ├── modules/
│   │   └── module1.py
│   └── inventory/
│       └── .../
├── README.md
├── roles/
│   ├── role1/
│   ├── role2/
│   └── .../
├── playbooks/
│   ├── files/
│   ├── vars/
│   ├── templates/
│   └── tasks/
└── tests/
```

Ansible Namespace is a way to organize and group related Ansible content, such as modules, plugins, and roles. Namespaces help avoid naming conflicts and provide a clear structure for Ansible content. Each namespace is associated with a specific collection, and the content within that namespace can be referenced using the namespace prefix.

The (FQCN) full qualified collection name of an Ansible module includes the namespace, collection, and module name. For example, the full qualified name of the apt module in the ansible.builtin collection is ansible.builtin.apt. "ansible.builtin" is the default namespace.

## 9 - Project: Deploy Nodejs application - Part 1

Create a droplet on digital ocean to install nodejs & deploy the application

Create hosts file with the following content:

```ini
server-ip ansible_ssh_private_key_file: ~/.ssh/id_ed25519 ansible_user: root
```

Create "deploy-node.yaml" playbook file with the following content:

```yaml
---
- name: Install Node & Npm
  hosts: server-ip
  tasks:
    - name: Update apt repo & cache
      # single line or multi line both work
      # apt: update_cache: yes force_apt_get: yes cache_valid_time: 3600
      apt:
        update_cache: yes
        force_apt_get: yes
        cache_valid_time: 3600
    - name: Install Nodejs & Npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Deploy Nodejs Application
  hosts: server-ip
  tasks:
    - name: Copy Nodejs application files to the server
      copy:
        src: /path/to/local/nodejs/app/nodejs-app-1.0.0.tgz
        dest: /root/app-1.0.0.tgz
    - name: Unpack the Nodejs application tar file
      unarchive:
        src: /root/app-1.0.0.tgz
        dest: /root/
        remote_src: yes
```

To generate the tar file for the nodejs app, run the following command in the local machine:

```bash
cd /path/to/local/nodejs/app
npm pack
```

Run the paybook with the following command:

```bash
ansible-playbook -i hosts deploy-node.yaml
```

SSH to the server & Check the server for the node js app

The "unarchive" module takes the ssource by default from the local machine and copies it to the remote server. So we can reduce the playbook as following with the same results:

```yaml
---
- name: Install Node & Npm
  hosts: server-ip
  tasks:
    - name: Update apt repo & cache
      # single line or multi line both work
      # apt: update_cache: yes force_apt_get: yes cache_valid_time: 3600
      apt:
        update_cache: yes
        force_apt_get: yes
        cache_valid_time: 3600
    - name: Install Nodejs & Npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Deploy Nodejs Application
  hosts: server-ip
  tasks:
    - name: Unpack the Nodejs application tar file
      unarchive:
        src: /path/to/local/nodejs/app/nodejs-app-1.0.0.tgz
        dest: /root/
```

## 10 - Project: Deploy Nodejs application - Part 2

Install the dependencies & run the nodejs app:

```yaml
---
- name: Install Node & Npm
  hosts: server-ip
  tasks:
    - name: Update apt repo & cache
      apt: update_cache: yes force_apt_get: yes cache_valid_time: 3600
    - name: Install Nodejs & Npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Deploy Nodejs Application
  hosts: server-ip
  tasks:
    - name: Unpack the Nodejs application tar file
      unarchive:
        src: /path/to/local/nodejs/app/nodejs-app-1.0.0.tgz
        dest: /root/
    - name: Install Nodejs application dependencies
      npm:
        path: /root/package
    - name: Start Nodejs application
      command:
        chdir: /root/package/app
        cmd: node server
      async: 1000
      poll: 0
    # register is used to save shell command output to a variable, so we can use it later in the playbook
    # debug for printing the output of the command to the console
    - name: Ensure Nodejs application is running
      shell: "ps aux | grep node server"
      register: app_status
    - debug: msg="Nodejs application is running: {{ app_status.stdout_lines }}"
```

Both "command" & "shell" modules can be used to run commands on the target server. The "command" module is more secure and faster than the "shell" module, but it does not support shell features such as pipes, redirects, and environment variable expansion.

"command" & "shell" modules are not stateful (idempotent), running above playbook multiple times will start multiple instances of the nodejs app. we can use conditional statements to check if the app is already running before starting it. 

## 11 - Project: Deploy Nodejs application - Part 3

For security best practices, we can create a new user to run the nodejs app instead of running it as root. 

```yaml
---
- name: Install Node & Npm
  hosts: server-ip
  tasks:
    - name: Update apt repo & cache
      apt: update_cache: yes force_apt_get: yes cache_valid_time: 3600
    - name: Install Nodejs & Npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Create a new user to run the Nodejs application
  hosts: server-ip
  tasks:
    - name: Create a new user
      user:
        name: nodejs
        comment: "User to run Nodejs application"
        group: admin

- name: Deploy Nodejs Application
  hosts: server-ip
  become: True
  become_user: nodejs
  tasks:
    - name: Unpack the Nodejs application tar file
      unarchive:
        src: /path/to/local/nodejs/app/nodejs-app-1.0.0.tgz
        dest: /home/nodejs/
    - name: Install Nodejs application dependencies
      npm:
        path: /home/nodejs/package
    - name: Start Nodejs application
      command:
        chdir: /home/nodejs/package/app
        cmd: node server
      async: 1000
      poll: 0
    # register is used to save shell command output to a variable, so we can use it later in the playbook
    # debug for printing the output of the command to the console
    - name: Ensure Nodejs application is running
      shell: "ps aux | grep node server"
      register: app_status
    - debug: msg="Nodejs application is running: {{ app_status.stdout_lines }}"
```

Run the playbook with the following command:

```bash
ansible-playbook -i hosts deploy-node.yaml
```


