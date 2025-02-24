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

## **How to Install Ansible on Windows WSL**

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
