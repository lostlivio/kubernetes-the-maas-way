# Kubernetes The MaaS Way
How to get Kubernetes up and running on bare-metal infrastructure managed by [Metal as a Service](https://maas.io/); referenced as` MaaS` for the remainder of the tutorial. 

This tutorial walks you through setting up Kubernetes on Bare-Metal that is managed by MaaS. It starts with the work done by [Kelsey Hightower - Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) and applies it to a MaaS environment. This guide is a quick reference to what it takes to stand-up a kubernetes cluster from scratch on infrastructure managed by MaaS.

> The results of this tutorial should not be viewed as production ready.

## Target Audience

The target audience for this tutorial is someone planning to support a production Kubernetes cluster on MaaS managed Bare-Metaland and wants to understand how everything fits together.

## Cluster Details

Kubernetes The MaaS Way guides you through bootstrapping a Kubernetes cluster with end-to-end encryption between components and RBAC authentication.

* [MAAS](https://github.com/maas/maas) 2.3.5
* [Kubernetes](https://github.com/kubernetes/kubernetes) 1.12.0
* [containerd Container Runtime](https://github.com/containerd/containerd) 1.2.0-rc.0
* [gVisor](https://github.com/google/gvisor) 50c283b9f56bb7200938d9e207355f05f79f0d17
* [CNI Container Networking](https://github.com/containernetworking/cni) 0.6.0
* [etcd](https://github.com/coreos/etcd) v3.3.9
* [CoreDNS](https://github.com/coredns/coredns) v1.2.2

## Labs

This tutorial assumes you have access to a [Maas](https://maas.io/) environment with 3 physical bare-metal nodes (1 Master Node and 2 Worker Nodes).

* [Prerequisites](docs/01-prerequisites.md)
* [Installing the Client Tools](docs/02-client-tools.md)
* [Provisioning Compute Resources](docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](docs/04-certificate-authority.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/05-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/06-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/07-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Provisioning Pod Network Routes](docs/11-pod-network-routes.md)
* [Deploying the DNS Cluster Add-on](docs/12-dns-addon.md)
* [Smoke Test](docs/13-smoke-test.md)
* [Cleaning Up](docs/14-cleanup.md)