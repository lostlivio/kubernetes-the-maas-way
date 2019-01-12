# Generating Kubernetes Configuration Files for Authentication

In this lab you will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), also known as kubeconfigs, which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

## Client Authentication Configs

In this section you will generate kubeconfig files for the `controller manager`, `kubelet`, `kube-proxy`, and `scheduler` clients and the `admin` user.

### Kubernetes Public IP Address

Each kubeconfig requires a Kubernetes API Server to connect to. 

> The public address was stored ealier in the **CONTROLLER_PUBLIC_ADDRESS** environment variable.

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Generate a kubeconfig file for each worker node:

```
for instance in worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-maas-way \
    --certificate-authority=secrets/ca.pem \
    --embed-certs=true \
    --server=https://${CONTROLLER_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=secrets/${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=secrets/${instance}.pem \
    --client-key=secrets/${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=secrets/${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-maas-way \
    --user=system:node:${instance} \
    --kubeconfig=secrets/${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=secrets/${instance}.kubeconfig
done
```

Files created results:

```
worker-1.kubeconfig
worker-2.kubeconfig
```

### The kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `kube-proxy` service:

```
{
  kubectl config set-cluster kubernetes-the-maas-way \
    --certificate-authority=secrets/ca.pem \
    --embed-certs=true \
    --server=https://${CONTROLLER_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=secrets/kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=secrets/kube-proxy.pem \
    --client-key=secrets/kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=secrets/kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-maas-way \
    --user=system:kube-proxy \
    --kubeconfig=secrets/kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=secrets/kube-proxy.kubeconfig
}
```

Files created results:

```
kube-proxy.kubeconfig
```

### The kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `kube-controller-manager` service:

```
{
  kubectl config set-cluster kubernetes-the-maas-way \
    --certificate-authority=secrets/ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=secrets/kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=secrets/kube-controller-manager.pem \
    --client-key=secrets/kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=secrets/kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-maas-way \
    --user=system:kube-controller-manager \
    --kubeconfig=secrets/kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=secrets/kube-controller-manager.kubeconfig
}
```

Files created results:

```
kube-controller-manager.kubeconfig
```


### The kube-scheduler Kubernetes Configuration File

Generate a kubeconfig file for the `kube-scheduler` service:

```
{
  kubectl config set-cluster kubernetes-the-maas-way \
    --certificate-authority=secrets/ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-secrets/scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=secrets/kube-scheduler.pem \
    --client-key=secrets/kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=secrets/kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-maas-way \
    --user=system:kube-scheduler \
    --kubeconfig=secrets/kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=secrets/kube-scheduler.kubeconfig
}
```

Files created results:

```
kube-scheduler.kubeconfig
```

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
{
  kubectl config set-cluster kubernetes-the-maas-way \
    --certificate-authority=secrets/ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=secrets/admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=secrets/admin.pem \
    --client-key=secrets/admin-key.pem \
    --embed-certs=true \
    --kubeconfig=secrets/admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-maas-way \
    --user=admin \
    --kubeconfig=secrets/admin.kubeconfig

  kubectl config use-context default --kubeconfig=secrets/admin.kubeconfig
}
```

Files created results:

```
admin.kubeconfig
```

## Distribute the Kubernetes Configuration Files

Copy the appropriate `kube-controller-manager` and `kube-scheduler` kubeconfig files to each controller instance:

```
scp secrets/admin.kubeconfig secrets/kube-controller-manager.kubeconfig secrets/kube-scheduler.kubeconfig ubuntu@$CONTROLLER_PUBLIC_ADDRESS:~/
```

Copy the appropriate `kubelet` and `kube-proxy` kubeconfig files to each worker instance:

```
scp secrets/worker-1.kubeconfig secrets/kube-proxy.kubeconfig ubuntu@$WORKER_1:~/
```

```
scp secrets/worker-2.kubeconfig secrets/kube-proxy.kubeconfig ubuntu@$WORKER_2:~/
```

Next: [Generating the Data Encryption Config and Key](06-data-encryption-keys.md)