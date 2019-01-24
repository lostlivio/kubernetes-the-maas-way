# Prerequisites

## Development Environment

This tutorial was created and tested on Ubuntu 18.04.

## MaaS (Metal as a Service) Environment Setup

This tutorial leverages [Metal as a Service](https://maas.io/) to manage the compute infrastructure required to bootstrap a Kubernetes cluster from the ground up. 

### Setup your hardware

You need one small server for MAAS and at least three addiitonal servers which can be managed with a BMC. It is recommended to have the MAAS server provide DHCP and DNS on a network the managed machines are connected to.

### Install Ubuntu Server

Install the base Ubunutu Operating system by downloading [Ubuntu Server 18.04 LTS](https://www.ubuntu.com/download/server?_ga=2.153985402.338891312.1544130217-65962038.1544130217) and follow the step-by-step installation instructions on your server.

SSH into your server.

```
ssh ubuntu@xxx.xxx.xxx.xxx
```

> xxx.xxx.xxx.xxx = <your.maas-server.ip>

Verify your Operating System version

```
lsb_release -a
```

> output

```
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.1 LTS
Release:	18.04
Codename:	bionic
```

### Install MaaS

Once Ubuntu is finished installing SSH into your Ubuntu server and install the MaaS application

```
sudo apt update && sudo apt install maas
```

### Create MaaS admin user

Create your MaaS admin credentials by typing.

```
sudo maas init
```

and then login with the credentials created using your browser to the MaaS UI located at http://<your.maas-server.ip>:5240/MAAS/


### Create MaaS admin user

Follow the instructions on screen to complete the installation of MAAS. I used the default values and completed the configuration for:

    * Region name (MAAS name)
    * Ubuntu archive, Ubuntu extra architectures
    * Ubuntu images
    * SSH keys (for currently logged in user)
    
> SSH keys added to an account will will be automatically uploaded to any provisioned node by the user account. 

### Turn on DHCP

Go to the “Subnets” tab, and select the VLAN for which you want to enable DHCP. From the “Take action” button select “Provide DHCP”.

    * Set the Rack controller that will manage DHCP.
    * Select the subnet where to create the DHCP Dynamic range on.
    * Fill in the details for the dynamic range.

> This tutorial uses a 192.168.1.0/24 range for the labs.

### Enlist and commission servers

Now MAAS is ready to enlist and commission machines. 

1. Setup servers connected to the same network lan as the MaaS server.
2. Set all the servers to PXE boot
3. Boot each machine once. You should see these machines appear in MAAS
4. If your machines do not have a IPMI based BMC, proceed to edit them and enter their BMC details
5. Select all the machines and “Commission” them using the “Take action” button
6. When machines have a “Ready” status you can start deploying

> output

View from **NODES** Tab

![MaaS Ready Image](/images/maas-ready.png)