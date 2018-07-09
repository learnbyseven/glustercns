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

