# Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab you will provision the compute resources required for running a secure and highly available Kubernetes cluster across a single [Availability Zones](https://docs.maas.io/2.4/en/manage-zones).

> Ensure you have 3 machines available as described in the [Prerequisites](01-prerequisites.md) lab and are in a **READY** state.

## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.


## Compute Instances

The compute instances in this lab will be provisioned with [Ubuntu Server](https://www.ubuntu.com/server) 18.04, which has good support for the [containerd container runtime](https://github.com/containerd/containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Prepare Nodes

Create three compute instances which will host the Kubernetes control plane:

1. Login to the MAAS UI at http://<your.maas.ip>:5240/MAAS/ and navigate to the **NODES** tab
2. Add a tag called `controller` to 1 machine. This is where we will install the kubernetes control plane.
3. Add a tag called `worker` to 2 additional machines. These will be used as the cluster worker nodes.
4. Place check marks next to the 3 machines selected in steps #1 and #2 then click the **TAKE ACTION** drop down and select **DEPLOY**
5. Select the Ubuntu Image with the **Ubuntu 18.04 LTS "Bionic Beaver"** version and **Default Kernel**
6. Click the **DEPLOY 3 MACHINES** button to start installing the Operating System.

### Verification

Once deployment is completed, you should see the following under the **NODES** tab. Each node should have **ON** in the **POWER** column and the OS Version 18.04 under the **STATUS** column.

![Compute Ready Image](/images/compute-ready.png)

### Prepare Development Environemnt

Create some environment variables that will be used during the following steps

```
MAAS_USERNAME=<insert_maas_username>
```

```
MAAS_API_KEY=<insert_maas_user_api_key>
```

> The API key can be retrieved from the MaaS UI by clicking on your account name at the top right hand corner of the screen.

```
MAAS_IP_ADDRESS=<insert_maas_ip_address>
```

Login to MaaS via the CLI 

```
maas login $MAAS_USERNAME http://localhost/MAAS/api/2.0/ $MAAS_API_KEY
```

### Kubernetes Controller IP Address

Get the controller IP address and populate the Controller IP Address into an environment variable

```
export CONTROLLER_PUBLIC_ADDRESS=$(maas $MAAS_USERNAME tag nodes controller | jq -r '.[].ip_addresses[0]')
```

> Note: If your servers have more than 1 interface, you will need to get fancy to iterate over the ip_addresses[index]. 

> Note: You can also just get the IP Address of each node by clicking on the name of machine under the **FQDN | MAC** column and navigating the the **INTERFACES** tab.

### Kubernetes Workers IP Addresses

Get the Worker IP addresses and populate the Worker Nodes IP Address into an environment variable

```
{
  i=1 
  for instance in $(maas $MAAS_USERNAME tag nodes worker | jq -r '.[].ip_addresses[0]'); do
    eval export WORKER_$i=$instance
   ((i++))
  done
}
```

> Note: If your servers have more than 1 interface, you will need to get fancy to iterate over the ip_addresses[index]

Each worker instance requires a pod subnet allocation from the Kubernetes cluster CIDR range. The pod subnet allocation will be used to configure container networking in a later exercise. The `pod-cidr` instance metadata will be used to expose pod subnet allocations to compute instances at runtime.

> The Kubernetes cluster CIDR range is defined by the Controller Manager's `--cluster-cidr` flag. In this tutorial the cluster CIDR range will be set to `192.168.2.0/24`, which supports 254 subnets.

### Verification

```
printenv | grep 'CONTROLLER_PUBLIC_ADDRESS\|WORKER_1\|WORKER_2'
```

>  output

```
CONTROLLER_PUBLIC_ADDRESS=192.168.2.88
WORKER_1=192.168.2.106
WORKER_2=192.168.2.59
```

> IPs above could be different based on DHCP configuration in step [Prerequisites](01-prerequisites.md)


## Testing SSH Access

SSH will be used to configure the controller and worker instances. 

Test SSH access to each of your compute instances:

```
ssh ubuntu@$CONTROLLER_PUBLIC_ADDRESS
```

```
ssh ubuntu@$WORKER_1
```

```
ssh ubuntu@$WORKER_2
```

You should see the following prompt for each of the nodes

### Verification

```
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-42-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Dec  6 22:43:52 UTC 2018

  System load:  0.0                Processes:              116
  Usage of /:   2.5% of 228.23GB   Users logged in:        0
  Memory usage: 1%                 IP address for enp0s25: x.x.x.x
  Swap usage:   0%


  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

3 packages can be updated.
3 updates are security updates.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permit


  ted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@nuc-01:
...

Type `exit` at the prompt to exit the compute instance:

```
ubuntu@nuc-01:~$ exit
```
> output

```
logout
Connection to x.x.x.x closed.
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)