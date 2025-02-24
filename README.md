# ansible-infra-automation

Automated infrastructure provisioning &amp; management with Ansible. Secure, scalable, and reusable playbooks. 🚀

> Ansible automates the management of remote systems and controls their desired state.

**Control node**: A system on which Ansible is installed. You run Ansible commands such as ansible or ansible-inventory on a control node.

**Inventory**: A list of managed nodes that are logically organized. You create an inventory on the control node to describe host deployments to Ansible.

**Managed node**: A remote system, or host, that Ansible controls.

## System Requirements

[Ansible Documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#control-node-requirements)

### Control Node

- UNIX-like OS (Linux, macOS, BSD, Windows via WSL)
- Python installed
- ❌ Native Windows (without WSL) not supported

### Managed Node

- Python required (for task execution)
- SSH access with a POSIX shell
- No need to install Ansible

✅ **Use `pipx` when:**

- Installing Python-based CLI tools (e.g., `ansible`, `black`, `poetry`).
- Keeping the global Python environment clean.

## **Windows WSL Control Node Setup**

### 1. Configure WSL

```bash
# Download ubuntu-22.04-server-cloudimg-amd64-root.tar.xz   from https://cloud-images.ubuntu.com/releases/22.04/release/
# Import that
wsl --import wsl-ubuntu "PATH TO WSL folder" PATH_TO_LINUX_IMAGE

#List all
wsl -l -v

# login to wsl
wsl -d <Distribution Name>

NEW_USER=<USERNAME>

# add user
useradd -m -G sudo -s /bin/bash "$NEW_USER"
passwd "$NEW_USER"  # Set the password

#configure default
tee /etc/wsl.conf <<_EOF
[user]
default=${NEW_USER}
_EOF

# Termiante
wsl --terminate <Distribution Name>

wsl -d <Distro name>

sudo apt update
sudo apt upgrade
sudo apt install software-properties-common
sudo apt install python3 python3-pip
# Or: curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
# python3 get-pip.py --user

echo "alias python='python3'" >> ~/.bashrc && source ~/.bashrc
```

### 2.1 Install Ansible - [Ubuntu installation instructions](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu)

1. Add the repo: `sudo add-apt-repository --y
es -update ppa:ansible/ansible`
2. Install: `sudo apt install ansible -y`
3. To confirm that Ansible is installed correctly, run: `ansible --version`

### 2.2 Install Ansible with pip

1. Ensure python is installed: `python3 -m pip -V`
2. Install ansible: `python3 -m pip install --user ansible`

## **How to build your inventory**

Ansible automates tasks on **managed nodes** using an **inventory** (list of hosts/groups).

### **Inventory Basics**

- Default file: `/etc/ansible/hosts`
- Custom inventory: `-i <path>` option
- Use **groups** to automate multiple hosts

#### **Example Inventory File**

```ini
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

- Headings in brackets are group names [Creating valid variable names](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#creating-valid-variable-names)

### Grouping

```ini
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
east:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
west:
  hosts:
    bar.example.com:
    three.example.com:
prod:
  children:
    east:
test:
  children:
    west:
```

### Organizing inventory in a directory

Ansible loads all inventory files from a directory if you **specify a folder** instead of a file.

**Loading Rules**

- Files are loaded in **ASCII order** (e.g., `01-hosts`, `02-extra`).
- Supports **`.ini`**, **`.yaml`**, and **`.json`** formats.
- Invalid files are ignored.

**How to Load a Directory as Inventory**

````bash
ansible -i /path/to/inventory_dir all --list-hosts
```
Use -i with the folder path to load all inventory files inside. 🚀

```bash
inventory/
openstack.yml # configure inventory plugin to get hosts from OpenStack cloud
dynamic-inventory.py # add additional hosts with dynamic inventory script
on-prem # add static hosts and groups
parent-groups # add static hosts and groups
````

You can target this inventory directory as follows

```bash
ansible-playbook example.yml -i inventory
```

### Assigning a variable to one machine: host variables

**In INI:**

```ini
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

**In YAML:**

```yaml
atlanta:
  hosts:
    host1:
      http_port: 80
      maxRequestsPerChild: 808
    host2:
      http_port: 303
      maxRequestsPerChild: 909
```

## **Managed node Setup**

Your managed nodes (the two Linux machines you wish to control) must meet a few basic requirements:

- **Python Installed**: Most modern Linux distributions have Python 3 pre-installed. If not, install it.
- **SSH Server Running**: Ansible uses SSH for communication.
- **Non-root User with sudo**: For security, create a dedicated user (or use an existing one) with sudo privileges.
- **Firewall Configuration**: Ensure SSH (port 22) is allowed.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 -y           # Usually pre-installed
sudo apt install openssh-server -y
```

### **The managed linux machine should have passwordlress SSH configured**

## Inventory File Setup

### Configuring SSH Key in ansible.cfg

If you prefer not to specify the SSH key location in your inventory file, you can set it globally in an Ansible configuration file (ansible.cfg):

```ini
[defaults]
inventory = inventory.ini
private_key_file = /home/youradmin/.ssh/id_rsa
```

### Create an Inventory File (e.g., inventory.ini):

```ini
[linux_hosts]
node1 ansible_host=192.168.1.101 ansible_user=yourusername
```

### Test Connectivity with the Ping Module

```bash
ansible linux_hosts -i inventory.ini -m ping
```

Must return:

```golang
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Verify SSH Key Usage via Ansible

```bash
ansible linux_hosts -i inventory.ini -m ping -vvv
```

### Testing Your Setup with an Ad-hoc Command

```bash
ansible linux_hosts -i inventory.ini -m command -a "uname -a"
```

---

## Creating and Running an Ansible Playbook

### What Is an Ansible Playbook?

An Ansible playbook is a YAML-formatted file that describes the desired state of your systems and the steps (tasks) required to reach that state. In a playbook, you define:

- **Hosts**: The target machines (as defined in your inventory) on which tasks will run.
- **Tasks**: Individual steps that can include running commands, cloning repositories, managing packages, configuring services, and more.
- **Modules**: Pre-built tools (like git, docker_container, etc.) that perform specific actions.
- **Variables**: Data that can be reused throughout the playbook.
- **Handlers**: Special tasks that are triggered by notifications from other tasks (for example, restarting a service when a configuration file changes).

### Sample Playbook: Testing a Public Repo Clone and Docker Image

Create a file named `deploy_test.yml` with the following content

### Explanation of the Playbook Commands

**Play Definition:**
The play begins with a name and specifies the target hosts (group managed from your inventory).
The become: yes allows privilege escalation (running commands as root), and gather_facts: yes collects system details from the managed node.

**Variables:**
Variables like repo_url, repo_dest, docker_image, and container_name are defined at the beginning. These variables make it easy to update the configuration for your environment.

**Task: Ensure Destination Directory Exists**
Uses the file module to create the directory if it doesn’t already exist. This prevents the git task from failing if the directory is missing.

**Task: Clone or Update the Public Repository**
The git module clones the repository from GitHub to the destination directory. It registers the result (stored in git_result) which can be used for debugging or notifying handlers.

**Task: Display Repository Clone Result**
Uses the debug module to print the output from the git task. This helps you verify that the repository was cloned or updated successfully.

**Task: Pull the Docker Image**
The docker_image module pulls the “hello-world” image from Docker Hub. This confirms that Docker is installed and working on your managed node.

**Task: Run the Docker Container**
The docker_container module starts a container from the pulled image. Since the hello-world container is designed to exit immediately after running, it is then stopped automatically.

**Task: Wait for the Container to Finish Running**
This ensures that the playbook waits until the container has completed its run before moving on.

**Task: Retrieve and Display Docker Container Logs**
The command module is used to run docker logs to fetch the output from the container. The logs are then displayed using the debug module to confirm that the Docker image ran as expected.

**Handler: Restart Container if Repository Updated**
Though not actively used in this example (you’d need to add a notification to the git task), this handler illustrates how you might restart a container if the code in your repository changes.

### Run the Playbook

```bash
ansible-playbook -i inventory.ini deploy_test.yml -vvv
```
