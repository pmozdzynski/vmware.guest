# VMware Guest Commissioning Role

An Ansible role for commissioning (provisioning) virtual machines on vSphere/vCenter. This role automates the creation, configuration, and management of VMs from templates with support for both DHCP and static IP configurations.

## Overview

This role provides a portable module for commissioning VMware guests on vSphere. It can:
- Create VMs from templates
- Configure hardware (CPU, memory, disks)
- Set up network configuration (DHCP or static IP)
- Deploy to clusters or specific ESXi hosts
- Manage VM folders
- Power on/off VMs
- Delete VMs when needed
- Wait for IP address assignment

## Prerequisites

1. **Ansible** installed (version 2.9 or higher recommended)
2. **Python dependencies** for VMware modules:
   ```bash
   pip install pyvmomi
   ```
3. **vCenter/vSphere access** with appropriate permissions:
   - Create/delete VMs
   - Create folders
   - Deploy from templates
   - Power on/off VMs
4. **VM Template** already created in vCenter

## Required Variables

The following variables must be defined when using this role:

### vCenter Connection
- `vcenter_hostname`: vCenter Server hostname or IP address
- `vcenter_username`: vCenter username
- `vcenter_password`: vCenter password
- `vcenter_datacenter`: Name of the datacenter
- `vcenter_cluster`: Name of the cluster (required unless `esxi_hostname` is specified)
- `vcenter_folder`: VM folder name where VMs will be created

### VM Configuration
- `vmguest_template`: Name of the VM template to use
- `vmguest_state`: Desired VM state (`poweredon`, `poweredoff`, or `absent`)
- `vmguest_id`: Guest OS identifier (e.g., `rhel8_64Guest`, `ubuntu64Guest`)
- `vmguest_memory`: Memory in MB
- `vmguest_num_cpu`: Number of CPUs
- `vmguest_disk_size`: Disk size in GB
- `vmguest_datastore`: Datastore name for VM disks

### Network Configuration
- `vmguest_vlan`: Network/VLAN name
- `vmguest_gateway`: Default gateway IP address
- `vmguest_network_type`: Network type - `dhcp` or `static` (default: `dhcp`)

### Static IP Configuration (required when `vmguest_network_type: static`)
- `vmguest_ip`: Static IP address for the VM
- `vmguest_netmask`: Subnet mask
- `nameserver1`: Primary DNS server
- `nameserver2`: Secondary DNS server
- `vmguest_domain`: Domain name

## Optional Variables

- `vmguest_name`: VM name (defaults to `inventory_hostname_short`)
- `vmguest_wait_for_ip_address`: Wait for IP address assignment (default: `true`)
- `vmguest_template_username`: Username for template access (default: `root`)
- `vmguest_template_password`: Password for template access
- `esxi_hostname`: Specific ESXi hostname (if deploying to a specific host instead of cluster)
- `ad_domain`: Active Directory domain (used for DNS lookup if not specified)
- `delete_machine`: Flag to allow VM deletion when `vmguest_state: absent`

## Usage Examples

### Basic Usage - Deploy VM with DHCP

```yaml
- hosts: localhost
  vars:
    vcenter_hostname: vcenter.example.com
    vcenter_username: administrator@vsphere.local
    vcenter_password: "{{ vault_vcenter_password }}"
    vcenter_datacenter: "Datacenter1"
    vcenter_cluster: "Cluster1"
    vcenter_folder: "Production"
    
    vmguest_name: "web-server-01"
    vmguest_template: "RHEL8-Template"
    vmguest_state: "poweredon"
    vmguest_id: "rhel8_64Guest"
    vmguest_memory: 4096
    vmguest_num_cpu: 2
    vmguest_disk_size: 50
    vmguest_datastore: "datastore1"
    
    vmguest_vlan: "VM Network"
    vmguest_gateway: "192.168.1.1"
    vmguest_network_type: "dhcp"
    vmguest_wait_for_ip_address: true

  roles:
    - vmware.guest
```

### Deploy VM with Static IP

```yaml
- hosts: localhost
  vars:
    vcenter_hostname: vcenter.example.com
    vcenter_username: administrator@vsphere.local
    vcenter_password: "{{ vault_vcenter_password }}"
    vcenter_datacenter: "Datacenter1"
    vcenter_cluster: "Cluster1"
    vcenter_folder: "Production"
    
    vmguest_name: "db-server-01"
    vmguest_template: "RHEL8-Template"
    vmguest_state: "poweredon"
    vmguest_id: "rhel8_64Guest"
    vmguest_memory: 8192
    vmguest_num_cpu: 4
    vmguest_disk_size: 100
    vmguest_datastore: "datastore1"
    
    vmguest_vlan: "VM Network"
    vmguest_network_type: "static"
    vmguest_ip: "192.168.1.100"
    vmguest_netmask: "255.255.255.0"
    vmguest_gateway: "192.168.1.1"
    nameserver1: "8.8.8.8"
    nameserver2: "8.8.4.4"
    vmguest_domain: "example.com"
    vmguest_wait_for_ip_address: true

  roles:
    - vmware.guest
```

### Deploy to Specific ESXi Host

```yaml
- hosts: localhost
  vars:
    vcenter_hostname: vcenter.example.com
    vcenter_username: administrator@vsphere.local
    vcenter_password: "{{ vault_vcenter_password }}"
    vcenter_datacenter: "Datacenter1"
    esxi_hostname: "esxi01.example.com"
    vcenter_folder: "Development"
    
    vmguest_name: "test-vm-01"
    vmguest_template: "Ubuntu-Template"
    vmguest_state: "poweredon"
    vmguest_id: "ubuntu64Guest"
    vmguest_memory: 2048
    vmguest_num_cpu: 2
    vmguest_disk_size: 30
    vmguest_datastore: "datastore1"
    
    vmguest_vlan: "VM Network"
    vmguest_gateway: "192.168.1.1"
    vmguest_network_type: "dhcp"

  roles:
    - vmware.guest
```

### Power Off a VM

```yaml
- hosts: localhost
  vars:
    vcenter_hostname: vcenter.example.com
    vcenter_username: administrator@vsphere.local
    vcenter_password: "{{ vault_vcenter_password }}"
    vcenter_datacenter: "Datacenter1"
    vcenter_cluster: "Cluster1"
    vcenter_folder: "Production"
    
    vmguest_name: "web-server-01"
    vmguest_state: "poweredoff"

  roles:
    - vmware.guest
```

### Delete a VM

```yaml
- hosts: localhost
  vars:
    vcenter_hostname: vcenter.example.com
    vcenter_username: administrator@vsphere.local
    vcenter_password: "{{ vault_vcenter_password }}"
    vcenter_datacenter: "Datacenter1"
    vcenter_cluster: "Cluster1"
    vcenter_folder: "Production"
    
    vmguest_name: "web-server-01"
    vmguest_state: "absent"
    delete_machine: true

  roles:
    - vmware.guest
```

### Using in a Playbook with Inventory

```yaml
# playbook.yml
- hosts: all
  vars:
    vcenter_hostname: vcenter.example.com
    vcenter_username: administrator@vsphere.local
    vcenter_password: "{{ vault_vcenter_password }}"
    vcenter_datacenter: "Datacenter1"
    vcenter_cluster: "Cluster1"
    vcenter_folder: "Production"
    
    vmguest_template: "RHEL8-Template"
    vmguest_state: "poweredon"
    vmguest_id: "rhel8_64Guest"
    vmguest_memory: 4096
    vmguest_num_cpu: 2
    vmguest_disk_size: 50
    vmguest_datastore: "datastore1"
    
    vmguest_vlan: "VM Network"
    vmguest_gateway: "192.168.1.1"
    vmguest_network_type: "dhcp"

  roles:
    - vmware.guest
```

```ini
# inventory.ini
[webservers]
web-server-01
web-server-02

[webservers:vars]
vmguest_memory=4096
vmguest_num_cpu=2
```

## VM States

- `poweredon`: Creates the VM (if it doesn't exist) and powers it on
- `poweredoff`: Powers off the VM (does not delete it)
- `absent`: Deletes the VM (requires `delete_machine` variable to be set)

## Network Configuration

### DHCP Mode
When `vmguest_network_type: dhcp`, the VM will receive its IP address automatically from DHCP. Only the gateway needs to be specified.

### Static IP Mode
When `vmguest_network_type: static`, you must provide:
- `vmguest_ip`: The static IP address
- `vmguest_netmask`: Subnet mask
- `vmguest_gateway`: Default gateway
- `nameserver1` and `nameserver2`: DNS servers
- `vmguest_domain`: Domain name

## Features

1. **Automatic Folder Management**: Creates VM folders if they don't exist
2. **VM Discovery**: Searches for existing VMs before creating new ones
3. **VM Movement**: Automatically moves VMs to the correct folder if needed
4. **IP Address Waiting**: Can wait for IP address assignment before completing
5. **ARP Announcement**: Pings the gateway to announce the VM's MAC address
6. **Template Customization**: Configures hostname and network settings during deployment

## Notes

- The role uses `validate_certs: no` by default. For production, consider using proper SSL certificates.
- VM names are automatically converted to uppercase for consistency.
- Hostnames in guest customization are set to lowercase.
- The role supports both cluster-based and ESXi host-specific deployments.
- When using static IP, ensure the IP address is not already in use.
- For security, store passwords in Ansible Vault rather than plain text.

## Troubleshooting

1. **VM creation fails**: Verify template exists and you have permissions
2. **IP address not assigned**: Check network configuration and ensure DHCP is available (for DHCP mode)
3. **Folder creation fails**: Verify datacenter name and folder permissions
4. **Template customization fails**: Ensure `vmguest_template_username` and `vmguest_template_password` are correct

## License

This role is provided as-is for VMware guest commissioning.

