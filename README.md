# glustercns
# Deploy Gluster CNS on 3.9 OCP 
## Perquisites 
- Suits converge environment
- 3 nodes with 3 Disks , /dev/xxx each node 
## Enabling Repos
- subscription-manager repos --enable={rh-gluster-3-client-for-rhel-7-server-rpms,rh-gluster-3-for-rhel-7-server-rpms,rh-gluster-3-nfs-for-rhel-7-server-rpms,rh-gluster-3-samba-for-rhel-7-server-rpms}
## Install Package:
- yum install glusterfs-fuse && yum update glusterfs-fuse

## Gluster Template glustercns.yaml 
ansible-playbook -i /root/ocpinstall/glustercnsdeploy /usr/share/ansible/openshift-ansible/playbooks/openshift-glusterfs/config.yml

## NOTE
Heketi secret is environment variable HEKETI_ADMIN_KEY="hfWqdIxUIaz8kOXMvd4XCFZjZywmwny7aqkKcq42qf0=" change into base 64 
example below:
echo -n "hfWqdIxUIaz8kOXMvd4XCFZjZywmwny7aqkKcq42qf0=" | base64
Secret 
restuser: admin 
key "aGZXcWRJeFVJYXo4a09YTXZkNFhDRlpqWnl3bXdueTdhcWtLY3E0MnFmMD0="

# Gluster CRS with block device provisioner 
### Images 
1. rhgs3/rhgs-gluster-block-prov-rhel7:latest
2. rhgs3/rhgs-volmanager-rhel7:latest

## Perquisites 
- On Gluster nodes 
- Container-Ready Storage requires the kernel-3.10.0-690.el7 version or higher to be used on the system.
- subscription-manager repos --enable={rh-gluster-3-client-for-rhel-7-server-rpms,rh-gluster-3-for-rhel-7-server-rpms,rh-gluster-3-nfs-for-rhel-7-server-rpms,rh-gluster-3-samba-for-rhel-7-server-rpms}
- yum install redhat-storage-server && gluster-block -y 
echo "dm_thin_pool" > /etc/modules-load.d/dm_thin_pool.conf
echo "target_core_user" > /etc/modules-load.d/target_core_user.conf
echo "dm_multipath" > etc/modules-load.d/dm_multipath.conf

#### systemctl start glusterd && systemctl enable glusterd
#### systemctl start gluster-blockd && systemctl enable gluster-blockd

#### ansible-playbook -i minimalocp3.9/minimal_v.01_host /usr/share/ansible/openshift-ansible/playbooks/openshift-glusterfs/config.yml

----------------------------------------------------------------------------------------------------------------------------------------[OSEv3:children]
masters
nodes
etcd
lb
nfs
glusterfs

### Set variables common for all OSEv3 hosts
[OSEv3:vars]
ansible_user=root
ansible_ssh_user=root
deployment_type=openshift-enterprise
openshift_release=v3.9
openshift_master_console_port=8443
openshift_master_api_port=8443
openshift_enable_unsupported_configurations=True

### GLUSTER CRS Container Ready Storage

openshift_storage_glusterfs_namespace=crs
openshift_storage_glusterfs_block_deploy=true
#openshift_storage_glusterfs_wipe=true
openshift_storage_glusterfs_is_native=false
openshift_storage_glusterfs_storageclass=true
openshift_storage_glusterfs_heketi_is_native=true
openshift_storage_glusterfs_heketi_executor=ssh
openshift_storage_glusterfs_heketi_ssh_port=22
openshift_storage_glusterfs_heketi_ssh_user=root
openshift_storage_glusterfs_heketi_ssh_sudo=false
openshift_storage_glusterfs_heketi_ssh_keyfile="/root/.ssh/id_rsa"
openshift_storage_glusterfs_block_host_vol_create=true
openshift_storage_glusterfs_block_host_vol_size=50
openshift_storage_glusterfs_block_storageclass=true
### Temp NFS Storage for registry
openshift_hosted_registry_storage_kind=nfs
openshift_hosted_registry_storage_access_modes=['ReadWriteMany']
openshift_hosted_registry_storage_nfs_directory=/exports
openshift_hosted_registry_storage_nfs_options='*(rw,root_squash)'
openshift_hosted_registry_storage_volume_name=registry
openshift_hosted_registry_storage_volume_size=10Gi

### COCKPIT


### ASB

### SERVICE CATALOG
#Prometheus deployment
openshift_hosted_prometheus_deploy=true

# Grafana deployment, requires Prometheus
openshift_hosted_grafana_deploy=true

### Load Balancer RADWARE ###
openshift_master_cluster_public_hostname=lbext.giriaddns.net
openshift_master_cluster_hostname=lb.giriaddns.net

### Authentication Provider settings.
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/ocpauth/users.htpasswd'}]

### Docker options
openshift_docker_options="--insecure-registry 172.30.0.0/16"
### Enable Network Time Protocol (NTP) to prevent masters and nodes in the cluster from going out of sync.
openshift_clock_enabled=true

### Default subdomain
openshift_master_default_subdomain=cloudapps.giriaddns.net

##Default nodes for router, registry and application pods
openshift_hosted_router_selector='region=infra'
openshift_registry_selector='region=infra'
osm_default_node_selector='region=primary'

## ROUTER & REGISTRY REPLICAS
openshift_hosted_router_replicas=1
openshift_hosted_registry_replicas=1

#openshift_disable_check=memory_availability,disk_availability,docker_storage
openshift_disable_check = docker_storage,memory_availability,disk_availability,docker_image_availability,package_version


##Configure kubeletArguments on nodes
openshift_node_kubelet_args={'image-gc-high-threshold': ['80'], 'image-gc-low-threshold': ['70']}

###Enable Openshift multi-tenant plugin
os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'

###Pod Network CIDR, default(10.0.0.0/14)
osm_cluster_network_cidr=10.0.0.0/14

###Service Network CIDR, default(172.30.0.0/16).
openshift_portal_net=172.30.0.0/16

##Audit Configuration
openshift_master_audit_config={"enabled": true, "auditFilePath": "/var/log/oscp-audit/oscp-audit.log", "maximumFileRetentionDays": 15, "maximumFileSizeMegabytes": 200, "maximumRetainedFiles": 10}

[masters]
mn1.giriaddns.net openshift_hostname=mn1.giriaddns.net
mn2.giriaddns.net openshift_hostname=mn2.giriaddns.net
mn3.giriaddns.net openshift_hostname=mn3.giriaddns.net

## Host group for etcd
[etcd]
mn1.giriaddns.net openshift_hostname=mn1.giriaddns.net
mn2.giriaddns.net openshift_hostname=mn2.giriaddns.net
mn3.giriaddns.net openshift_hostname=mn3.giriaddns.net


## Host group for nodes
[nodes]
mn1.giriaddns.net openshift_hostname=mn1.giriaddns.net schedulable=true openshift_node_labels="{'region': 'infra', 'zone':'default', 'cluster':'stag'}"
mn2.giriaddns.net openshift_hostname=mn2.giriaddns.net schedulable=true openshift_node_labels="{'region': 'primary', 'zone':'default', 'cluster':'stag'}"
mn3.giriaddns.net openshift_hostname=mn3.giriaddns.net schedulable=true openshift_node_labels="{'region': 'primary', 'zone':'default', 'cluster':'stag'}"

## NFS Server
[nfs]
lb.giriaddns.net

[lb]
lb.giriaddns.net

### GLUSTER FS
[glusterfs]
mn1.giriaddns.net glusterfs_ip=192.168.124.200 glusterfs_devices='[ "/dev/sdb" ]'
mn2.giriaddns.net glusterfs_ip=192.168.124.201 glusterfs_devices='[ "/dev/sdb" ]'
mn3.giriaddns.net glusterfs_ip=192.168.124.202 glusterfs_devices='[ "/dev/sdb" ]'
----------------------------------------------------------------------------------------------------------------------------------------














TASK [openshift_storage_glusterfs : Add iptables allow rules] *******************************************************************
skipping: [mn1.giriaddns.net] => (item={u'port': u'2222/tcp', u'service': u'glusterfs_sshd'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'24007/tcp', u'service': u'glusterfs_management'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'24008/tcp', u'service': u'glusterfs_rdma'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'49152-49251/tcp', u'service': u'glusterfs_bricks'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'2222/tcp', u'service': u'glusterfs_sshd'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'24010/tcp', u'service': u'glusterblockd'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'24007/tcp', u'service': u'glusterfs_management'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'3260/tcp', u'service': u'iscsi-targets'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'24008/tcp', u'service': u'glusterfs_rdma'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'111/tcp', u'service': u'rpcbind'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'49152-49251/tcp', u'service': u'glusterfs_bricks'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'24010/tcp', u'service': u'glusterblockd'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'2222/tcp', u'service': u'glusterfs_sshd'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'3260/tcp', u'service': u'iscsi-targets'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'24007/tcp', u'service': u'glusterfs_management'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'111/tcp', u'service': u'rpcbind'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'24008/tcp', u'service': u'glusterfs_rdma'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'49152-49251/tcp', u'service': u'glusterfs_bricks'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'24010/tcp', u'service': u'glusterblockd'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'3260/tcp', u'service': u'iscsi-targets'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'111/tcp', u'service': u'rpcbind'})

TASK [openshift_storage_glusterfs : Remove iptables rules] **********************************************************************

TASK [openshift_storage_glusterfs : Add firewalld allow rules] ******************************************************************
skipping: [mn1.giriaddns.net] => (item={u'port': u'2222/tcp', u'service': u'glusterfs_sshd'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'24007/tcp', u'service': u'glusterfs_management'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'24008/tcp', u'service': u'glusterfs_rdma'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'2222/tcp', u'service': u'glusterfs_sshd'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'49152-49251/tcp', u'service': u'glusterfs_bricks'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'24007/tcp', u'service': u'glusterfs_management'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'24010/tcp', u'service': u'glusterblockd'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'24008/tcp', u'service': u'glusterfs_rdma'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'3260/tcp', u'service': u'iscsi-targets'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'2222/tcp', u'service': u'glusterfs_sshd'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'24007/tcp', u'service': u'glusterfs_management'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'24008/tcp', u'service': u'glusterfs_rdma'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'49152-49251/tcp', u'service': u'glusterfs_bricks'})
skipping: [mn1.giriaddns.net] => (item={u'port': u'111/tcp', u'service': u'rpcbind'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'49152-49251/tcp', u'service': u'glusterfs_bricks'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'24010/tcp', u'service': u'glusterblockd'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'24010/tcp', u'service': u'glusterblockd'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'3260/tcp', u'service': u'iscsi-targets'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'3260/tcp', u'service': u'iscsi-targets'})
skipping: [mn2.giriaddns.net] => (item={u'port': u'111/tcp', u'service': u'rpcbind'})
skipping: [mn3.giriaddns.net] => (item={u'port': u'111/tcp', u'service': u'rpcbind'})


TASK [Set GlusterFS install 'Complete'] *****************************************************************************************
ok: [-------------------------------------------------------------------------]

PLAY RECAP **********************************************************************************************************************
------------------------------------------------------------------------- : ok=4    changed=0    unreachable=0    failed=0
lb.giriaddns.net           : ok=1    changed=0    unreachable=0    failed=0
localhost                  : ok=14   changed=0    unreachable=0    failed=0
mn1.giriaddns.net          : ok=105  changed=30   unreachable=0    failed=0
mn2.giriaddns.net          : ok=26   changed=0    unreachable=0    failed=0
mn3.giriaddns.net          : ok=26   changed=0    unreachable=0    failed=0


INSTALLER STATUS ****************************************************************************************************************
Initialization             : Complete (0:06:24)
GlusterFS Install          : Complete (0:05:48)








