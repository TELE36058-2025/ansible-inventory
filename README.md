# ansible-inventory

# Understanding Ansible Inventory

## Introduction

One key component of Ansible is its inventory. Ansible's inventory lets you organise and define the targets you will automate. Whether you're automating a small network or a large-scale deployment, a solid inventory setup—and knowing how to work with it—makes all the difference.

In this guide, we'll explore the two types of Ansible inventory—static and dynamic—and their role in network automation. We will also examine the different inventory formats (INI and YAML), learn how to configure dynamic inventories using plugins for Sources of Truth such as NetBox, and discover how to filter your inventory for greater control over which host you are automating!

## What is an Ansible Inventory?

An Ansible inventory is a collection of hosts and the necessary data to allow Ansible to connect to them. In other words, it defines the "who"!

For example, an inventory includes IP addresses, authentication credentials, and other configuration details needed for Ansible to execute tasks on the target systems. This information enables Ansible to interact with the correct devices and apply the desired automation defined in a Playbook.

## Types of Ansible Inventory

Ansible supports two primary types of inventory: Static and Dynamic.

### Static Inventory

A static inventory is a manually maintained file that lists all hosts and their groupings. Typically written in INI or YAML format, a static inventory is ideal for environments where host configurations are stable and require infrequent modifications.

### Dynamic Inventory

A dynamic inventory retrieves host and group information at runtime from external systems such as a Source of Truth like NetBox, configuration management databases, or APIs. By leveraging scripts or plugins, dynamic inventories automatically generate an up-to-date list of the required inventory information for Ansible to successfully execute tasks on the target systems. This makes them well-suited for large-scale, dynamic environments where inventory data needs to reflect frequent changes.

### Comparison: Static vs. Dynamic Inventory

| Feature | Static Inventory | Dynamic Inventory |
|---------|-----------------|-------------------|
| Setup | Manually created and maintained | Uses scripts or plugins to fetch hosts dynamically |
| Updates | Requires manual updates | Automatically updates as infrastructure changes |
| Use Case | Small, stable environments | Large, dynamic environments or those leveraging a Source of Truth like NetBox |
| Scalability | Limited scalability | Highly scalable |
| Examples | Flat files in INI or YAML format | NetBox, InfraHub plugins |

## Inventory Formats in Ansible

### Static Inventory – INI Format

The INI format is straightforward and easy to use. Hosts and groups are organized using sections defined by square brackets ([]).

```ini
# Inventory file for managing network devices

# Group: Core Routers
[core_routers]
router1 ansible_host=192.168.0.1 ansible_network_os=ios 
router2 ansible_host=192.168.0.2 ansible_network_os=ios

# Group: Access Switches
[access_switches]
switch1 ansible_host=192.168.1.1 ansible_network_os=nxos
switch2 ansible_host=192.168.1.2 ansible_network_os=nxos

# Group: Firewalls
[firewalls]
fw1 ansible_host=192.168.2.1 ansible_network_os=asa
fw2 ansible_host=192.168.2.2 ansible_network_os=asa

# Nested Group: All Network Devices
[network_devices:children]
core_routers
access_switches
firewalls

# Group Variables for Switches
[access_switches:vars]
vlans=10,20,30
```

### Static Inventory – YAML Format

```yaml
all:
  children:
    core_routers:
      hosts:
        router1:
          ansible_host: 192.168.0.1
          ansible_network_os: ios
        router2:
          ansible_host: 192.168.0.2
          ansible_network_os: ios
    access_switches:
      hosts:
        switch1:
          ansible_host: 192.168.1.1
          ansible_network_os: nxos
        switch2:
          ansible_host: 192.168.1.2
          ansible_network_os: nxos
      vars:
        vlans: [10, 20, 30]
    firewalls:
      hosts:
        fw1:
          ansible_host: 192.168.2.1
          ansible_network_os: asa
        fw2:
          ansible_host: 192.168.2.2
          ansible_network_os: asa
    network_devices:
      children:
        - core_routers
        - access_switches
        - firewalls
```

## Dynamic Inventory: NetBox Integration

### Installing and Setting Up the NetBox Ansible Collection

1. Install the NetBox Ansible Collection:
```bash
ansible-galaxy collection install netbox.netbox
```

2. Set Environment Variables:
```bash
export NETBOX_API="https://netbox.example.com"
export NETBOX_TOKEN="YOUR_NETBOX_API_TOKEN"
```

### Example Configuration

Create a configuration file (nb_inventory.yaml):

```yaml
plugin: ansible.netcommon.netbox
api_endpoint: "https://netbox.example.com"
token: "{{ lookup('env','NETBOX_TOKEN') }}"
validate_certs: False
query_filters:
  site: HQ
group_by:
  - device_role
  - device_type
```

### Key Configuration Parameters

- **token**: API token sourced from environment variables
- **validate_certs**: Boolean for SSL certificate validation
- **config_context**: Enables inclusion of NetBox configuration contexts
- **group_by**: Defines how devices are organized
- **group_names_raw**: Strips prefixes from group names
- **compose**: Creates custom variables from NetBox attributes
- **query_filters**: Filters for retrieving specific devices

## Ansible Inventory Command-Line Tool

### Common Options

#### ansible-inventory --graph

Creates a tree structure visualization of the inventory:

```
@all:
  |--@core_routers:
  |  |--router1
  |  |--router2
  |--@access_switches:
  |  |--switch1
  |  |--switch2
  |--@firewalls:
  |  |--fw1
  |  |--fw2
```

#### ansible-inventory --list

Outputs all host and group information in JSON format.

## Filtering Ansible Playbook Execution with --limit

### Common Usage Patterns

| Use Case | Command Example | Description |
|----------|----------------|-------------|
| Single host | `--limit router1` | Runs on router1 only |
| Specific group | `--limit access_switches` | Runs on access_switches group |
| Multiple hosts | `--limit "router1:fw1"` | Runs on router1 and fw1 |
| Multiple groups | `--limit "core_routers:firewalls"` | Runs on both groups |
| Exclude group | `--limit "!firewalls"` | Runs on all except firewalls |
| Exclude host | `--limit "!router1"` | Runs on all except router1 |
| Wildcard match | `--limit "router*"` | Runs on hosts starting with router |
| Combined filters | `--limit "access_switches:!switch2"` | Runs on group except switch2 |
