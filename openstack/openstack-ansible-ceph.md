There're 5 step installing Openstack with openstack-ansible:
  1. Prepare deployment host
  2. Prepare target host
  3. Configure deployment
  4. Run playbooks
  5. Verify Openstack operations
  6. Configure DVR_SNAT

## 1. Prepare deployment host

![image](https://github.com/pahrialms/openstack-ansible-ceph/assets/82088448/cb508776-2961-4dce-a06c-c10ffad5956d)
source : https://docs.openstack.org/project-deploy-guide/openstack-ansible/latest/deploymenthost.html

Configure Ubuntu 
* run only on deployment host (ex: controller001)
```
apt update
apt dist-upgrade
apt install build-essential git chrony openssh-server python3-dev sudo
```

Install the source and depedencies
```
git clone -b master https://opendev.org/openstack/openstack-ansible /opt/openstack-ansible
cd /opt/openstack-ansible
scripts/bootstrap-ansible.sh
```

## 2. Prepare target host

![image](https://github.com/pahrialms/openstack-ansible-ceph/assets/82088448/6e4da354-45ad-4c2b-a4ac-69a648598229)
source: https://docs.openstack.org/project-deploy-guide/openstack-ansible/latest/targethosts.html

Configure Ubuntu
* run on all node
```
apt update
apt dist-upgrade
apt install bridge-utils debootstrap openssh-server tcpdump vlan python3
```

Configure networking 

![image](https://github.com/pahrialms/openstack-ansible-ceph/assets/82088448/f1a96136-83ca-4670-8377-8ac0ec6ab192)


## 3. Configure deployment

![image](https://github.com/pahrialms/openstack-ansible-ceph/assets/82088448/1eccbe0f-60c8-4612-9a24-64b9d0f7ed22)
source : https://docs.openstack.org/project-deploy-guide/openstack-ansible/latest/configure.html

Copy openstack directory configuration to etc, then there're 2 file configuration that we need changes to make it suite with our environment :
  1. `openstack_user_config.yml`. This file is heavily commented with details about the various options.
  2. `user_variables.yml`, file to configure global and role specific deployment options. The file contains some example variables and comments but you can get the full list of variables in each role’s specific documentation.

```
cd /opt/openstack-ansible
cp -R etc/openstack_deploy /etc/

nano openstack_user_config.yml

---
cidr_networks:
  management: 172.20.20.0/24
  tunnel: 172.20.21.0/24
  storage: 172.16.16.0/24

used_ips:
  - "172.20.20.1,172.20.20.50"
  - "172.20.21.1,172.20.21.50"
  - "172.20.22.1,172.20.22.50"
  - "172.16.16.1,172.16.16.50"

global_overrides:
  internal_lb_vip_address: 172.20.20.50
  #
  # The below domain name must resolve to an IP address
  # in the CIDR specified in haproxy_keepalived_external_vip_cidr.
  # If using different protocols (https/http) for the public/internal
  # endpoints the two addresses must be different.
  #
  external_lb_vip_address: 172.20.22.50
  management_bridge: "br-mgmt"
  provider_networks:
    - network:
        container_bridge: "br-mgmt"
        container_type: "veth"
        container_interface: "eth1"
        ip_from_q: "management"
        type: "raw"
        group_binds:
          - all_containers
          - hosts
        is_management_address: true
    - network:
        container_bridge: "br-vxlan"
        container_type: "veth"
        container_interface: "eth10"
        ip_from_q: "tunnel"
        type: "vxlan"
        range: "1:1000"
        net_name: "vxlan"
        group_binds:
          - neutron_openvswitch_agent
    - network:
        container_bridge: "br-vlan"
        container_type: "veth"
        type: "vlan"
        range: "101:200"
        net_name: "physnet1"
        group_binds:
          - neutron_openvswitch_agent
    - network:
        container_bridge: "br-storage"
        container_type: "veth"
        container_interface: "eth2"
        ip_from_q: "storage"
        type: "raw"
        group_binds:
          - glance_api
          - cinder_api
          - cinder_volume
          - nova_compute

###
### Infrastructure
###

# galera, memcache, rabbitmq, utility
shared-infra_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

# repository (apt cache, python packages, etc)
repo-infra_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

coordination_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13
# load balancer
# Ideally the load balancer should not use the Infrastructure hosts.
# Dedicated hardware is best for improved performance and security.
haproxy_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

###
### OpenStack
###

# keystone
identity_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

# cinder api services
storage-infra_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

# glance
# The settings here are repeated for each infra host.
# They could instead be applied as global settings in
# user_variables, but are left here to illustrate that
# each container could have different storage targets.
image_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

# placement
placement-infra_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

# nova api, conductor, etc services
compute-infra_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13


# horizon
dashboard_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

# neutron api
network-infra_hosts:
  ms-controller001:
    ip: 172.20.20.11
  ms-controller002:
    ip: 172.20.20.12
  ms-controller003:
    ip: 172.20.20.13

# neutron agents (L3, DHCP, etc)
network-agent_hosts:
 ms-compute001:
   ip: 172.20.20.14
 ms-compute002:
   ip: 172.20.20.15

# nova hypervisors
compute_hosts:
 ms-compute001:
   ip: 172.20.20.14
 ms-compute002:
   ip: 172.20.20.15


# cinder volume hosts (NFS-backed)
# The settings here are repeated for each infra host.
# They could instead be applied as global settings in
# user_variables, but are left here to illustrate that
# each container could have different storage targets.
storage_hosts:
  ms-compute001:
    ip: 172.20.20.14
    container_vars:
      cinder_backends:
        limit_container_types: cinder_volume
        lvm:
          volume_group: cinder-volumes
          volume_driver: cinder.volume.drivers.lvm.LVMVolumeDriver
          volume_backend_name: LVM_iSCSI
          iscsi_ip_address: "172.16.16.14"
  ms-compute002:
    ip: 172.20.20.15
    container_vars:
      cinder_backends:
        limit_container_types: cinder_volume
        lvm:
          volume_group: cinder-volumes
          volume_driver: cinder.volume.drivers.lvm.LVMVolumeDriver
          volume_backend_name: LVM_iSCSI
          iscsi_ip_address: "172.16.16.15"
---
```

```
nano user_variables.yml

---
debug: false
openstack_host_specific_kernel_modules:
  - name: "openvswitch"
    pattern: "CONFIG_OPENVSWITCH="
    group: "network_hosts"
neutron_plugin_type: ml2.ovs.dvr
neutron_ml2_drivers_type: "flat,vlan,vxlan"
haproxy_keepalived_external_vip_cidr: "172.20.22.50/32"
haproxy_keepalived_internal_vip_cidr: "172.20.20.50/32"
haproxy_keepalived_external_interface: br-ext
haproxy_keepalived_internal_interface: br-mgmt
neutron_provider_networks:
  network_flat_networks: "*"
  network_types: "vlan"
  network_vlan_ranges: "physnet1:101:200"
  network_mappings: "physnet1:br-provider"
neutron_plugin_base:
 - router
 - metering
glance_default_store: rbd
install_method: source
ceph_mons:
  - 172.16.16.11
  - 172.16.16.12
  - 172.16.16.13
glance_ceph_client: glance
glance_rbd_store_pool: images
## copy content from /etc/ceph/ceph.conf to in
ceph_conf_file: |
  [global]
  fsid = d625899c-a20f-11ee-a99d-25b34172f724
  mon_host = [v2:172.16.16.11:3300/0,v1:172.16.16.11:6789/0] [v2:172.16.16.12:3300/0,v1:172.16.16.12:6789/0] [v2:172.16.16.13:3300/0,v1:172.16.16.13:6789/0]
cinder_ceph_client: cinder
cinder_backends:
  rbd:
  volume_driver: cinder.volume.drivers.rbd.RBDDriver
  rbd_pool: volumes
  rbd_ceph_conf: /etc/ceph/ceph.conf
  rbd_store_chunk_size: 8
  volume_backend_name: rbd
  rbd_user: "{{ cinder_ceph_client }}"
  rbd_secret_uuid: "{{ cinder_ceph_client_uuid }}"
  report_discard_supported: true

nova_libvirt_images_rbd_pool: vms
---
```
Configure Service credentials
```
cd /opt/openstack-ansible
./scripts/pw-token-gen.py --file /etc/openstack_deploy/user_secrets.yml
```

## 4. Run playbooks

![image](https://github.com/pahrialms/openstack-ansible-ceph/assets/82088448/e66999ad-cf20-47df-ba03-b20159dd2073)
source: https://docs.openstack.org/project-deploy-guide/openstack-ansible/latest/run-playbooks.html

The installation process requires running three main playbooks:

  1. The `setup-hosts.yml` Ansible foundation playbook prepares the target hosts for infrastructure and OpenStack services, builds and restarts containers on target hosts, and installs common components into containers on target hosts.
  2. The `setup-infrastructure.yml` Ansible infrastructure playbook installs infrastructure services: Memcached, the repository server, Galera and RabbitMQ.
  3. The `setup-openstack.yml` OpenStack playbook installs OpenStack services, including Identity (keystone), Image (glance), Block Storage (cinder), Compute (nova), Networking (neutron), etc.

Check playbooks syntaks
```
openstack-ansible setup-hosts.yml --syntax-check
openstack-ansible setup-infrastructure.yml --syntax-check
openstack-ansible setup-openstack --syntax-check
```

Run playbooks to install openstack
* it will take times around 40-60+ mnt

```
openstack-ansible setup-hosts.yml
openstack-ansible setup-infrastructure.yml
openstack-ansible setup-openstack.yml
```

## 5. Verify Openstack operations

![image](https://github.com/pahrialms/openstack-ansible-ceph/assets/82088448/76e68627-c0c0-4518-aa73-104fecb79f14)
https://docs.openstack.org/project-deploy-guide/openstack-ansible/latest/verify-operation.html

Verify API
```
lxc-ls -f
# lxc-ls -f | grep utility
ms-controller001_utility_container-a0f9a0e5        RUNNING 1         onboot, openstack 10.0.3.226, 172.20.20.69                -    false

lxc-attach -n ms-controller001_utility_container-a0f9a0e5
/# source /root/openrc
/# openstack endpoint list
```

## 6. Configure DVR_SNAT 

Because in my environment I'm not using dedicated network node for traffic SNAT, so I must configure compute node to handle this traffic
* execute on all compute node
```
cd /etc/neutron/
nano l3_agent.ini
---
[DEFAULT]
agent_mode = dvr_snat
---
systemctl restart neutron-l3-agent.service
```




