# openstack-ansible-ceph
This documentation how to install openstack with openstack-ansible and using ceph as storage distribution

### Network Cofiguration :

| Host              | BondM          | Bond0.10        | Bond0.30        | Bond0.40        |Bond1.20        | Bond1            |
|-------------------|----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
|                   | Management     | br-mgmt          | br -vxlan      | br-ext          |br-storage       | FIP Network     |
| ms-controller002 | 172.18.2.12/24 | 172.20.20.12/24 | 172.20.21.12/24 | 172.20.22.12/24 |  172.16.16.12/24 |                 |
| ms-controller003 | 172.18.2.13/24 | 172.20.20.13/24 | 172.20.21.13/24 | 172.20.22.13/24 | 172.16.16.13/24 |                  |
| ms-compute001     | 172.18.2.14/24 | 172.20.20.14/24 | 172.20.21.14/24 | 172.20.22.14/24 | 172.16.16.14/24 |                 |
| ms-compute002     | 172.18.2.15/24 | 172.20.20.15/24 | 172.20.21.15/24 | 172.20.22.15/24 | 172.16.16.15/24 |                 |
| ms-compute003     | 172.18.2.16/24 | 172.20.20.16/24 | 172.20.21.16/24 | 172.20.22.16/24 | 172.16.16.16/24 |                 |


Install Openstack with openstack-ansible with storage distribution Ceph

1. [Install ceph with cephadm](https://github.com/pahrialms/kolla-ansible-with-ceph/blob/main/ceph/cephadm.md)
2. [Install openstack with openstack-ansible](https://github.com/pahrialms/openstack-ansible-ceph/blob/main/openstack/openstack-ansible-ceph.md)
3. [Create resource Openstack](https://github.com/pahrialms/integrate-openstack-kube/blob/main/openstack/create-resource-openstack.md)
