# Ansible-DevOps-Essentials
## If I'll find any other feature to be useful - it'll surely be added!
## How it works?
Ansible is a tool for automating and managing configuration.

It allows for connecting to multiple managed nodes \(anything that needs to be configured\) from control node \(node with Ansible that manages its execution\) and running tasks that will configure the desired state of the node \(declarative approach\). Ansible is agentless \(there is no need for running agents on managed nodes\) and idempotent \(tasks are safe to execute multiple times - there will be the same result\).

Managed nodes are defined in [Inventory](#inventory) file and tasks are defined in [Playbook](#playbooks) files in YAML syntax.

Configuration management is not so important right now because of containers \(that are small, pre-prepared environments\) but still can be useful for some cases.
## Playbooks
Playbooks define desired state of a configured node. They are defined as a sequence of tasks that need to be executed on selected nodes, under provided conditions. \
Tasks can be looped to execute multiple times for different values. \
Tags can be used to run only specific Playbook tasks \(with --tags flag\). \
Blocks can be used to group tasks with common logic, e.g. to handle errors.

Example of Playbook:
```
---
- name: <playbook_name>
  hosts: <inventory_group>
  become: <yes_or_no> # Use sudo/root
  vars:
    <var_name>: <var_value>

  tasks:
    - name: <task_name>
      <module_or_command>:
        <module_option>: <option_value>
      when: <condition>

    - name: <task_name>
      <module_or_command>:
        <module_option>: "{{ item }}"
      loop:
        - <value1>
        - <value2>
        - <value3>
      tags: <tag_names>

    - block
      - name: <task_name>
        <module_or_command>:
          <module_option>: <option_value>

      - name: <task_name>
        <module_or_command>:
          <module_option>: <option_value>
    rescue:
      - name: <task_name>
        <module_or_command>:
          <module_option>: <option_value>
```
### Modules
Modules are ready-to-use scripts that run specific commands \(e.g. apt, service, copy\). You can develop them on your own but you can use available ones - full list can be found at [Module Index](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html).
### Handlers
Handlers are tasks that are called only when they are notified by other tasks \(e.g. reset service after changes in configuration\). They are called only once, even if they will be notified multiple times.

Example of Handler:
```
---
- name: <playbook_name>
  hosts: <inventory_group>
  become: <yes_or_no> # Use sudo/root
  vars:
    <var_name>: <var_value>

  tasks:
    - name: <task_name>
      <module_or_command>:
        <module_option>: <option_value>
      notify: <handler_name> # Call handler after change

  handlers:
    - name: <handler_name>
      <module_or_command>:
        <module_option>: <option_value>
```
### Variables
Variables are simple key-value data that can be dynamically used in playbooks.

They can be defined by \(in order of precedence\):
* **-e** - extra vars, variable values from command line option
* **vars** - defined in playbook/task
* **host_vars/ and group_vars/ directories** - YAML/JSON files inside directories, that define variables for specific host or group

Variables can be of types:
* **regular** - simple variables defined by user
* **facts** - automatically collected metadata about managed nodes
* **magic variables** - built-in Ansible variables
* **registered variables** - task results saved to variable, e.g. below
```
- name: <task_name>
  command: <command>
  register: <variable_name>

- name: Print variable
  debug:
    var: <variable_name>.stdout
```
### Best practises
* **Name and comment tasks properly** to maintain readability
* **Use Ansible Vault to encrypt sensitive data** - ```ansible-vault encrypt vars/secrets.yml```
* **Dry-run with --check --diff**
* **Use ansible-lint to check syntax**
## Inventory
Inventory is a list of hosts and groups which defines nodes that will be managed, how to group them and which variable values use with them.

Example of Inventory:
```
[<group_name>]
<host>
<host> ansible_port=<port> # Custom SSH port

[<group_name>:vars] # Group variables
<variable_name>=<variable_value>

[<parent_group_name>:children]  # Group that contains other groups
<group_name>
```
### Best practises
* **Structure inventories** for different environments \(e.g. dev, test, prod\), static and dynamic, etc.
* Like with Playbooks - **Use Ansible Vault to encrypt sensitive data**
* **Use --graph and --list to visualize inventories** - ```ansible-inventory -i inventories/ --graph```, ```ansible-inventory -i inventories/inventory --list```
## Dynamic Inventory
Dynamic Inventory is a mechanism that allows for generating managed nodes list based on provided infrastructure data instead of relying on static files. It is useful for cloud or container \(e.g. Kubernetes\) environments or when using infrastructure managing tools \(e.g. Terraform\) where infrastructure is ephemeral and scalable \(e.g. infrastructure was recreated or new hosts added by scaling\).

You can use ready-made scripts \(e.g. aws_ec2.py\) or create your own that gets data from infrastructure API and generates formatted JSON file that will be recognised as inventory file or use ready-made plugins \(e.g. amazon.aws.aws_ec2, kubernetes.core.k8s\) that are recommended.

Example of Dynamic Inventory configuration using amazon.aws.aws_ec2 plugin:
```
plugin: amazon.aws.aws_ec2
regions:
  - <region_name>
filters:
  tag:<tag_name>: <tag_value> # Filter instances with tag
keyed_groups:
  - key: <key> # Create host groups based on e.g. tag
    prefix: <prefix>
compose:
  ansible_host: public_ip_address # Use public IP to connect
```
To use Dynamic Inventory above you must install boto3 and botocore \(```pip install boto3 botocore```\) and configure ```AWS_ACCESS_KEY_ID``` and ```AWS_SECRET_ACCESS_KEY``` environment variables.

To optimize using infrastructure API use cache:
```
# ansible.cfg
[inventory]
cache = yes
cache_plugin = jsonfile
cache_timeout = <seconds>
```
## Roles
