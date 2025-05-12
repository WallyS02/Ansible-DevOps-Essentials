# Ansible-DevOps-Essentials
## If I'll find any other feature to be useful - it'll surely be added!
## How it works?
Ansible is a tool for automating and managing configuration.

It allows for connecting to multiple managed nodes \(anything that needs to be configured\) from control node \(node with Ansible that manages its execution\) and running tasks that will configure the desired state of the node. Ansible is agentless \(there is no need for running agents on managed nodes\) and idempotent \(tasks are safe to execute multiple times - there will be the same result\).

Managed nodes are defined in [Inventory](#inventory) file and tasks are defined in [Playbook](#playbooks) files in YAML syntax.

Configuration management is not so important right now because of containers \(that are small, pre-prepared environments\) but still can be useful for some cases.
## Playbooks
## Inventory
## Variables and Facts
## Roles
## Modules
## Dynamic Inventory
## Handlers
