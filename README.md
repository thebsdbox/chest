# Shack
The Plunder Shack is virtual environment for developing and testing deployment tooling
![](https://github.com/plunder-app/shack/blob/master/image/shack.png?raw=true)

## Usage

This section will detail the expected usage of `shack`.

### Set Environment Configuration

You can use `shack` to build and configure the environment, it will generate a example configuration that should work out of the box in most use-cases:

```
./shack example > shack.yaml

$ cat shack.yaml 
bridgeAddress: 192.168.1.1/24
bridgeName: plunder
nicMacPrefix: 'c0:ff:ee:'
nicPrefix: plunderVM
```

This configuration specifies the name of the bridge and it's address that it will use, it also specifies the prefix of mac addresses create for VMs along with the prefix of the tap addresses. 

### Configure Environment

With the configuration in place we can use `shack` to set up all of the networking infrastructure we require for our virtual machines to live on. We can validate the environment with:

```
$ sudo ./shack network check
shack Networking configuration
WARN[0000] Link not found  
```
As we can see, we've not created our `shack` environment yet, so we'll create and check again (we will also inspect the networking with `ip link`):

```
$ sudo ./shack network create
shack Networking configuration

$ sudo ./shack network check
shack Networking configuration

$ ip addr show plunder
3: plunder: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default 
    inet 192.168.1.1/24 brd 192.168.1.255 scope global plunder
```

### Start a virtual machine

With the network in place we will start our virtual machine(s):

```
$ sudo ./shack vm start
shack VM configuration
Network Device:	plunderVM-b5987b
VM MAC Address:	c0:ff:ee:b5:98:7b
VM UUID:	b5987b
```
The UUID is used to identify a particular VM, it also is part of it's assigned MAC address and finally is used to communicate with the virtual machine through a socket `/tmp/qmp-<UUID>`

We can inspect our virtual machine(s) networking:

```
$ brctl show
bridge name	bridge id		STP enabled	interfaces
plunder		8000.faeed3bc56d0	no		plunderVM-b5987

$ ip addr show plunderVM-b5987
4: plunderVM-b5987: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master plunder state UP group default qlen 1000
    link/ether fa:ee:d3:bc:56:d0 brd ff:ff:ff:ff:ff:ff
```

### Stop a virtual machine

To stop a virtual machine `shack` will communicate with the qmp socket `/tmp/qmp-<UUID>`, we can stop our recently started Virtual machine with the command:

```
$  sudo ./shack vm stop --id b5987b
```

**Note** the `stop` command will wait for 10 seconds before "terminating" the virtual machine.
**Additional Note**, the `/tmp/qmp-<UUID>` will be removed, however the `<UUID>.qcow2` will need removing manually.

## Provisioning tooling

In order to use `shack` as a demonstrating environment then configure your tooling to use the `bridgeName` as the interface to broadcast on and use the `bridgeAddress` as the server address/gateway for new machines.
