# Red Hat OpenStack 13 for NFVi Lab
## How to
### Use those templates
Those templates have been developed using the infra-as-code principles and crafted from production ones.
The structure somehow diverges from the standard TripleO/OSPd baseline adding deterministic and reproducible environment limiting at the minimum the randomness.
Knowledge about how those have been developed can be found in the [Red Hat OpenStack Platform Advanced Configuration chapter](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/13/html/advanced_overcloud_customization/)
### Deploy an NFVi Lab
To deploy a platform, the undercloud must be installed. No wrapper is provided for this scope. The undercloud configuration is under versioning control.
Then moving into the script folder, in numerical order all must be executed.
### Apply Incremental Changes
Simply re-execute the `004-deploy.sh` wrapper.
### Use those templates as a baseline for a new NFVi platform
Usually changes between clouds can be found in the following files
 - The `003-create_update_docker_osp_images.sh` and `004-deploy.sh` -> To include the right templates
 - 10-commons-parameters.yaml -> For the root password and cloud name
 - 20-network-environment.yaml -> Network details (VLAN, Subnets, allocation ranges, etc)
 - 40-fencing.yaml -> MAC addreses and IPMI details
 - 60-openstack-neutron-custom-configs.yaml -> Neutron Bridge Mapping, DHCP, and other Neutron-related configuration
 - 70-ovs-dpdk-sriov.yaml -> Compute role types configuration (CPU-Partitioning, Kernel Args, PMD Design, SR-IOV Design etc)
## Folder structure
```
├── deployment_wrappers	=> Wrapper directory including NFVi deployment and initialization scripts (e.g. Docker images, import baremetal environment, etc)
├── overcloud		=> Overcloud main directory for custom templates and environment configuration
│   ├── environments	=> Templates and configurations for the platform (network, SSL, storage, tuning, and extraconfig hooks mapping)
│   ├── extraconfig	=> Pre and post-puppet customization hooks templates and scripts
│   │   ├── post-config	=> Post-puppet templates (e.g. configure Heat caching)
│   │   └── pre-config	=> (PLACE HOLDER) empty
│   ├── firstboot	=> Very first boot cutomization template (e.g. SSH root password)
│   └── nic-configs	=> Per role physical NIC mapping to bonds, vlan tags, subnets, and routes
├── scripts		=> Script directory including some various tools
├── undercloud		=> Undercloud directory including undercloud configuration and baremetal environment
└── validation		=> Script directory including basic validation script to funcionally verify the platform once deployed/updated
```
## File structure
```
├── README.md						=> This Readme
├── deployment_wrappers
│   ├── 001-create_update_osp_images.sh			=> Wrapper to create and update the overcloud RHEL images
│   ├── 002-import_instackenv_nodes.sh			=> Wrapper to import/update the virtual/physical server into Ironic
│   ├── 003-create_update_docker_osp_images.sh		=> Wrapper to download and update the container images from the Red Hat DCN - active subscription required
│   └── 004-deploy.sh					=> Wrapper to deploy the platform and also to apply incremental changes
├── overcloud
│   ├── environments					=> Main environment folder with all the templates describing the deployment
│   │   ├── 10-commons-parameters.yaml			=> Common environment parameters (e.g. TimeZone, root password, SSH config, and DNS cloud name)
│   │   ├── 20-network-environment.yaml			=> Network configuration file defining subnets, vlans, allocation rangies, bonding setup, fixed IPs, Virtual IPs, etc
│   │   ├── 30-storage-environment.yaml			=> Storage configuration file for all the services
│   │   ├── 40-fencing.yaml				=> Hash with server Ethernet MAC address, IPMI credential, and fencing mechanism
│   │   ├── 50-keystone-admin-endpoint.yaml		=> Expose Keystone Admin API over external network
│   │   ├── 60-openstack-neutron-custom-configs.yaml	=> Specific Neutron configuration such as MTU size, bridge mapping, security group firewall driver etc
│   │   ├── 65-openstack-nova-custom-configs.yaml	=> Specific Nova configuration such as Multipathd, scheduler filters etc
│   │   ├── 70-ovs-dpdk-sriov.yaml			=> SR-IOV and OVS-DPDK Compute configuration such as Tuned profile, CPU Partitioning, Huge Pages, kernel args etc
│   │   ├── 99-extraconfig.yaml				=> Puppet Extra configuration for Pre and Post deployment hooks, tuning (oslo, memcached, rabbit, haproxy, mysql) etc
│   │   └── 99-server-blacklist.yaml			=> Blacklist nodes useful for upgrade purpose
│   ├── extraconfig
│   │   ├── post-configi				=> Post deployment template folder currently only used by controller role to enable Heat cache
│   │   │   ├── controller.yaml
│   │   │   └── scripts
│   │   │       └── heatcache.sh
│   │   └── pre-confi					=> (PLACE HOLDER) - Pre deployment template folder currently NOT usedg
│   │       ├── compute_deterministic.yaml
│   │       └── scripts
│   │           └── qemu-cron-job
│   ├── firstboot
│   │   └── first-boot.yaml				=> First boot template for root SSH password and Disk Wipe (used by Ceph, not required here)
│   ├── nic-configs
│   │   ├── compute-dual-ovsdpdk.yaml			=> Compute with OVS-DPDK with NICs on both NUMA nodes
│   │   ├── compute-dual-sriov.yaml			=> Compute with SR-IOV with NICs on both NUMA nodes
│   │   ├── compute-ovsdpdk-sriov.yaml			=> Compute with OVS-DPDK NIC on NUMA0 and SR-IOV NIC on NUMA1
│   │   ├── compute-ovsdpdk.yaml			=> Compute with OVS-DPDK NIC only on NUMA0
│   │   ├── compute-sriov.yaml				=> Compute with SR-IOV NIC only on NUMA0
│   │   └── controller.yaml				=> Controller NIC Mapping
│   ├── nodes-info.yaml					=> Node info such as quantity per node
│   ├── overcloud_images.yaml				=> Docker images for the Overcloud
│   └── roles-data.yaml					=> Roles template used to define additional compute roles
├── scripts
│   ├── token_perf.sh					=> Generate Keystone tokens in a while true loop to measure the Keystone Token issue API response time
│   └── token_perf_uc.sh				=> Same as above only this one is for the Undercloud
├── undercloud
│   ├── custom_hieradata.yaml				=> Undercloud Puppet's Hieradata overwrite to improve for API responvness
│   ├── db_tuning.sh					=> Undercloud MariaDB tuning script - Must be executed after any `openstack undercloud [install|upgrade]`
│   ├── instackenv.json					=> Physical and Virtual Hardware definition
│   └── undercloud.conf					=> Undercloud main configuration file
└── validation
    ├── basic_ovs-dpdk
    │   ├── create_gre.sh				=> Create simple validation virtual environment with GRE tenant networks
    │   ├── create_vxlan.sh				=> Create simple validation virtual environment with VXLAN tenant networks
    │   └── teardown.sh					=> Virtual environment teardown
    └── basic_sriov
        ├── create_pf.sh				=> Create simple validation virtual environment with SR-IOV PF (also called PCI-Passthrough)
        ├── create_vf.sh				=> Create simple validation virtual environment with SR-IOV VF
        └── teardown.sh					=> Virtual environment teardown
```
## Userful Knowledge Base
- [Configure Docker to use a proxy with or without authentication](https://access.redhat.com/solutions/1377973)
- [Misalignment Nova Placement API](https://ask.openstack.org/en/question/115081/openstack-queen-instance-creation-error-no-valid-host-was-found/)
- [How to configure HTTP Proxy for Red Hat Subscription Management](https://access.redhat.com/solutions/57669)
- [Ironic node using dracclient goes to clean failed with "Unfinished config jobs found"](https://bugzilla.redhat.com/show_bug.cgi?id=1534551)
- [Nova instance provision fails with host is not mapped to any cell in Red Hat OpenStack Platform](https://access.redhat.com/solutions/3268111)
- [Starting 2nd VM on same NUMA socket is causing huge latency during startup of second VM](https://bugzilla.redhat.com/show_bug.cgi?id=1678810#c39)
## Outstanding Deterministic Performance and Real-Time Issues
- [OVS-DPDK bridges should be DOWN](https://bugzilla.redhat.com/show_bug.cgi?id=1628227)
- [[RHOSP13.0.6][OVS-DPDK] CPU steal in PMD thread imapcting physical Rxq performance](https://bugzilla.redhat.com/show_bug.cgi?id=1734368)
- [8 vCPU guest need max latency < 20 us with stress](https://bugzilla.redhat.com/show_bug.cgi?id=1690543)
- [\<stats period='10'/> of memballoon device cause high latency spike for KVM-RT guest](https://bugzilla.redhat.com/show_bug.cgi?id=1701509)
- [[RFE] Performance Monitoring Unit management for real time guest](https://bugzilla.redhat.com/show_bug.cgi?id=1646397)
- [spurious ktimersoftd wake ups increases latency (rhel-rt 7)](https://bugzilla.redhat.com/show_bug.cgi?id=1550584)
- [add compiler optimization option to openvswitch rpm package for better ovs-dpdk performance](https://bugzilla.redhat.com/show_bug.cgi?id=1633719)
- [[RHOSP10][NFV][SRIOV][ISOLCPUS] High SoftIRQ interrupt in guest vcpu0 / Isolated KVM Core in NFVi](https://bugzilla.redhat.com/show_bug.cgi?id=1667911)
- [dropped packets noticed in dpctl/show in balance-slb with passive lacp bonds](https://bugzilla.redhat.com/show_bug.cgi?id=1701825)
- [IRQs silently spilling over onto isolated cores](https://bugzilla.redhat.com/show_bug.cgi?id=1714686)
- [do not raise timer softirq unconditionally](https://bugzilla.redhat.com/show_bug.cgi?id=1730016)
