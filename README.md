# Ansible-role-TKGm
An ansible role that installs Tanzu TKGm management and workload clusters

## Scope
This role is a personal effort to deploy TKGm _2.5.0_ into a vSphere environment.  
Since I've been using this a lot to spin up clusters in my homelab I decided to release this publicly.
_Important:_ This role is best effort and is not officially backed by VMware by Broadcom.

## Supported features
- Air-gapped deployment.
- The use of NSX-T alb for L4 loadbalancing.

## Limitations
- If you deploy an airgapped instance, this role assumes you already have all images and tanzu plugins in the local repository.
- Currently the role only supports LDAPS and not OIDC (coming soon).
- Upgrading clusters is not possible yet.
- The role assumes you are running this on a host that is able to reach all cluster networks. ie: The tanzu jumpbox/ admin mgmt server.

## Using the role
1. Specify the cluster FQDN in your inventory.
2. Set the appropriate variables in your host and/or group vars.
3. Create a playbook that references this role and inventory. Disable fact gathering for hosts!
4. run the playbook against the TKG host(s).

See [This walkthrough](Walkthrough.md) for a detailed example and walkthrough of how I use this role.

## Variables
There are quite a lot of variables for this role. Below is the list with what they mean.
All these variables can be found in `defaults/main.yml` and will either have a default value or will be empty if there's no default value applicable.

Most of these variables are directly linked to the TKG configuration variables found on https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.4/tkg-deploy-mc/config-ref.html

```yaml
tanzu_version: v2.5.0 # Check the Release notes for correct versions.
tanzu_cli_version: # This is automatically filled in from the version matrix in the vars directory.
package_version:  # This is automatically filled in from the version matrix in the vars directory.
kubeconfig_location: /tmp # A location on the filesystem to store kubeconfigs.
default_binary_directory: /usr/local/bin # Change if your OS uses another directory.
```
Below are default variables that are applicable for both management and workload clusters. You probably want to  overwrite these in either group or host vars depending on your architecture.  
For example: Workload clusters on another vcenter/datacenter than the management cluster.
```yaml
## Shared cluster vars
delete_cluster: false # Keep this at false and overwrite with extra-vars to true to delete a cluster.
cluster_type: # is either: mgmt|workload
cluster_name: "{{ inventory_hostname_short }}" # the cluster FQDN in your hosts file.
cluster_plan: dev # Either dev (1CP, 1 worker) or prod (3 CP, 3 workers ).
os_name: "ubuntu" # Either ubuntu or photon.
cluster_cidr: "10.117.64.0/19" # Default k8s cidrs, change if these already exist in your environment.
service_cidr: "10.117.96.0/19" # Default k8s cidrs, change if these already exist in your environment.
vsphere_server: # FQDN of your vcenter server.
vsphere_username: # vCenter username.
vsphere_password: # vCenter user password.
vsphere_datacenter: # vSphere datacenter.
vsphere_resource_pool: # vSphere RP.
vsphere_datastore: # vSphere datastore.
vsphere_folder: # vSphere folder where the cluster nodes will be deployed in.
vsphere_network: # The vSphere network your cluster nodes will attach to.
vsphere_tls_thumbprint: # The TLS thumbprint of your vCenter instance.
vsphere_insecure: false
vsphere_ssh_authorized_key: # a dedicated SSH public key that will be injected into the capv user on every node for troubleshooting.
vsphere_control_plane_endpoint: "{{ inventory_hostname }}" # Can be either an IP or FQDN, see the TKG docs for more info.
control_plane_machine_count: 1 # Default controlplane node count.
vsphere_control_plane_num_cpus: 2 # CPU count for all controlplane nodes.
vsphere_control_plane_disk_gib: 20 # Disk size for all controlplane nodes.
vsphere_control_plane_mem_mib: 4096 # MEmory size for all controlplane nodes.
worker_machine_count: 1 # Default worker node count.
vsphere_worker_num_cpus: 2 # CPU count for all worker nodes.
vsphere_worker_disk_gib: 20 # Disk size for all worker nodes.
vsphere_worker_mem_mib: 4096 # MEmory size for all worker nodes.
tkg_custom_image_repository: # When deploying in an airgapped, provide the repo URL (ie: myharbor.mydomain.com/tkg). 
tkg_custom_image_repository_skip_tls_verify: false # If using selfsigned certs on your registry, you can ignore the certs here.
tkg_custom_image_repository_ca_certificate: # Provide a base64 encoded PEM certificate of your private CA that signed your registry.
ipam_enabled: false # Enable or disable node IPAM to replace DHCP for nodes.
ipam_ip_pool_gateway: # When using IPAM: provate the default gateway here.
ipam_ip_pool_addresses: [] # A list of ip pools to use for node IPAM.
ipam_ip_pool_subnet_prefix: 24 # The subnet prefix of the subnet that's used for node IPAM.
ipam_nameservers: # Comma separated list of DNS servers
```

The following variables are specifically for management clusters. This is mostly pinniped authentication variables and NSX-T ALB variables.  
See the TKG documentation for details on these variables: 
```yaml
# Mgmt cluster vars
ldap_bind_dn:
ldap_bind_password:
ldap_group_search_base_dn: 
ldap_host: 
ldap_root_ca_data_b64:
avi_enabled: false
avi_ca_data_b64: 
avi_cloud_name: 
avi_control_plane_network: 
avi_control_plane_network_cidr: 
avi_controller: 
avi_data_network: 
avi_data_network_cidr: 
avi_labels: 
avi_management_cluster_control_plane_vip_network_cidr: 
avi_management_cluster_control_plane_vip_network_name: 
avi_management_cluster_service_engine_group: 
avi_management_cluster_vip_network_cidr: 
avi_management_cluster_vip_network_name: 
avi_password:
avi_username: 
avi_service_engine_group:
```

These are some specific workload cluster variables, including the packages.
```yaml
# Workload cluster vars
mgmt_cluster_name: "" # Define the name of the mgmt cluster that manages this referenced workload cluster.
kubevip_enabled: false # Enable or disable kubevip as Loadbalancer provider.
kubevip_range: [] # List Format: 10.0.0.1-10.0.0.255
cluster_namespace: "default"
cluster_ipam_type: InClusterIPPool # Change to GlobalInClusterIPPool when you specified a global pool

# Packages vars
packages_namespace: "kapp-packages"
packages: [] # The role dynamically grabs the last version of the package. Overwrite as below code in comment to choose a specific version.
#  - name: cert-manager
#    version: 1.12.2+vmware.1-tkg.1
#    values_file: False
#  - name: contour
#    version: 1.25.2+vmware.1-tkg.1
#    values_file: True
```
