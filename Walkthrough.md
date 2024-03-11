# TKGm role walkthrough
This is a brief walkthrough showing how I implement this role in my Ansible environment.

## Ansible directory overview
```
ansible_root_directory:
  inventory/
    hosts
    group-vars/
      tkg/
        tkg.yml
    host_vars/
      mgmt-cluster-1.mylab.local/
        tkg.yml
      workload-cluster-1.mylab.local/
        tkg.yml
  roles/
    tkg
  playbook.yml
```

### Inventory
Add your clusters to your inventory. As with all Ansible inventories, it highly depends on your environment. In this example, we'll keep it simple:

```yaml
>>> hosts
all:
  children:
    tkg:
      children:
        management:
          hosts:
            mgmt-cluster-1.mylab.local:
        workload:
          hosts:
            workload-cluster-1.mylab.local:
```

### Host and Group roles
Create a host_vars directory for every cluster and a group_vars directory for your cluster group.  

In my case, I only need to create group_vars for the tkg group. This variable file contains all the vSphere scoped vars since all my clusters end up in the same vSphere environment. If you have different vSphere enviroments like an Edge situation, you will need to make sure your inventory and group_vars reflect that situation.  
```yaml
>>> inventory/group_vars/tkg
vsphere_server: 
vsphere_username: 
vsphere_password: 
vsphere_datacenter: 
os_name: 
vsphere_resource_pool: 
vsphere_datastore: 
vsphere_folder: 
vsphere_network:
vsphere_tls_thumbprint:
vsphere_ssh_authorized_key: 
kubeconfig_location: 
tkg_custom_image_repository_ca_certificate: 
tkg_custom_image_repository: 
ipam_nameservers:
```
Once you have your group_vars set, you can specify the remaining variables for your clusters in their respective host_vars.  
```yaml
>>> inventory/host_vars/mgmt-cluster-1.mylab.local
cluster_type: mgmt
ipam_enabled: true
ipam_ip_pool_gateway:
ipam_ip_pool_addresses: 
  - ""
ipam_ip_pool_subnet_prefix: 
```
```yaml
>>> inventory/host_vars/workload-cluster-1.mylab.local
cluster_type: workload
mgmt_cluster_name: mgmt-cluster-1
kubevip_enabled: true
kubevip_range: 
  - ""
ipam_enabled: true
ipam_ip_pool_gateway: 
ipam_ip_pool_addresses: 
  - ""
ipam_ip_pool_subnet_prefix: 
packages: 
  - name: cert-manager
    values_file: False
  - name: contour
    values_file: True
```

### Playbook
Lastly, create your playbook file:
```yaml
>>> tkg.yml
- hosts: tkg
  gather_facts: false
  roles:
    - tkg
```

## Running the playbook
Once all variables have been set, you can run the playbook against your clusters.  
``` ansible-playbook tkg.yaml -l mgmt-cluster-1.mylab.local```