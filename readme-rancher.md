# Running OpenStack on Rancher

This repo is a fork of [Kolla](https://github.com/stackforge/kolla) to try running OpenStack on [Rancher](https://github.com/rancherio/rancher).
  
## Install OpenStack on Rancher

Get a 2-node Rancher cluster working. 
RAM: master - 1GB, node01 - 8GB, node02 - 4GB

```
NODE_01_IP=104.236.220.193  # just for example
NODE_02_IP=104.236.99.202   # same
```

## node01 containers

### bootstrap-files
- Image: `imikushin/compose` (source: https://github.com/imikushin/compose)
- Entrypoint: `bash`
- Volumes: 
  * `/usr/local/src:/usr/local/src`
  * `/var/run/docker.sock:/var/run/docker.sock`

Launch the container, then run in the container's console:
```bash
cd /usr/local/src/
git clone https://github.com/imikushin/kolla.git
cd kolla/
./tools/genenv
```

`./tools/genenv` will print (we'll need `$MY_IP` value):
```
MY_IP=172.17.0.7
MY_DEV=eth0
Please customize your FLAT_INTERFACE to a different network then your
main network. The FLAT_INTERFACE is used for inter-VM communication.
the FLAT_INTERFACE should not have an IP address assigned.
```

Replace `$MY_IP` value with `$NODE_01_IP` in `./compose/openstack.env` and `./openrc`:
```bash
sed -i.bak "s/${MY_IP}/${NODE_01_IP}/" ./compose/openstack.env
sed -i.bak "s/${MY_IP}/${NODE_01_IP}/" ./openrc
```

### start-rabbitmq
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/rabbitmq.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed. 

`compose_rabbitmq_1` container will pop up.  

### start-mariadb
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/mariadb.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_mariadbdata_1` container will land in stopped state.  
`compose_mariadbapp_1` container will pop up.  

### start-keystone
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/keystone.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_keystone_1` container will pop up.  

### start-glance-api-registry
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/glance-api-registry.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_glanceregistry_1` container will pop up.  
`compose_glanceapi_1` container will pop up.  

### start-nova-api-conductor-scheduler
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/nova-api-conductor-scheduler.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_novaconductor_1` container will pop up.  
`compose_novaapi_1` container will pop up.  
`compose_novascheduler_1` container will pop up.  

### start-nova-compute
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/nova-compute.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_computedata_1` container will land in stopped state.  
`compose_libvirt_1` container will pop up.  
`compose_novacompute_1` container will pop up.  

### start-neutron-server
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/neutron-server.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_neutronserver_1` container will pop up.  

### start-neutron-agents
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/neutron-agents.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_neutronagents_1` container will pop up.  

### start-heat-api-engine
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/heat-api-engine.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_heatapi_1` container will pop up.  
`compose_heatengine_1` container will pop up.  

### start-horizon
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/horizon.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_horizon_1` container will pop up.  


## node02 containers

### bootstrap-files
- Image: `imikushin/compose`
- Entrypoint: `bash`
- Volumes: 
  * `/usr/local/src:/usr/local/src`
  * `/var/run/docker.sock:/var/run/docker.sock`

Launch the container, then run in the container's console:
```bash
cd /usr/local/src/
git clone https://github.com/imikushin/kolla.git
cd kolla/
./tools/genenv
```

`./tools/genenv` will print (we'll need `$MY_IP` value):
```
MY_IP=172.17.0.7
MY_DEV=eth0
Please customize your FLAT_INTERFACE to a different network then your
main network. The FLAT_INTERFACE is used for inter-VM communication.
the FLAT_INTERFACE should not have an IP address assigned.
```

Replace `$MY_IP` value with `$NODE_01_IP` (not a typo: we want node01 IP here) 
in `./compose/openstack.env` and `./openrc`:
```bash
sed -i.bak "s/${MY_IP}/${NODE_01_IP}/" ./compose/openstack.env
sed -i.bak "s/${MY_IP}/${NODE_01_IP}/" ./openrc
```

### start-nova-compute
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/nova-compute.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_computedata_1` container will land in stopped state.  
`compose_libvirt_1` container will pop up.  
`compose_novacompute_1` container will pop up.  

### start-neutron-agents
- Image: `imikushin/compose`
- Commmand: `-f /usr/local/src/kolla/compose/neutron-agents.yml up -d`
- Volumes From: `bootstrap-files`

After this container has stopped, it's no longer needed.

`compose_neutronagents_1` container will pop up.  


## Demo environment

At this point you'll have a (well, sort of) working OpenStack installation.

1.  Go to `http://${NODE_01_IP}/horizon`
2.  Authenticate with `admin` / `steakfordinner`

### Create a VM image

1.  Go to `http://${NODE_01_IP}/horizon/project/images/`
2.  Hit "Create Image" and fill out the form:
  - Name: cirros-0.3.3-x86_64
  - Image Location: http://cdn.download.cirros-cloud.net/0.3.3/cirros-0.3.3-x86_64-disk.img
  - Format: QCOW2
  - Public: true
3.  Hit "Create Image".

### Create networks

1.  Go to `http://${NODE_01_IP}/horizon/project/networks/`
2.  Hit "Create Network" and fill out the form:
  - Network Name: network0
  - Subnet Name: subnet0
  - Network Address: 192.168.0.0/24
  - IP Version: IPv4
3.  Hit "Create".
4.  Hit "Create Network" and fill out the form:
  - Network Name: network1
  - Subnet Name: subnet1
  - Network Address: 192.168.1.0/24
  - IP Version: IPv4
5.  Hit "Create".

Visit `http://${NODE_01_IP}/horizon/project/network_topology/` to see your networks visualized. 
You can also create routers on this page.

### Create a VM instance

1.  Go to `http://${NODE_01_IP}/horizon/project/instances/`
2.  Hit "Create Network" and fill out the form:
  - Instance Name: vm1
  - Instance Boot Source: Boot from image
  - Image Name: cirros-0.3.3-x86_64
  - Selected Networks: network1 (drag from Available networks)
3.  Hit "Launch".

Again, visit `http://${NODE_01_IP}/horizon/project/network_topology/` for a nice picture of 
your VM attached to your network.

