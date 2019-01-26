# Provisioning a CA and Generating TLS Certificates

Provision PKI Infrastructure, bootstrap a Certificate Authority, and generate TLS certificates for all components.

## Certificate Authority

Create a Folder to store all the secrets

```
mkdir secrets
```

Create the CA configuration file and the CA certificate signing request::

```
cat > secrets/ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF
```

```
cat > secrets/ca-csr.json << EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Seattle",
      "O": "Kubernetes",
      "OU": "WA",
      "ST": "Washington"
    }
  ]
}
EOF
```

Generate the CA certificate and private key:

```
cfssl gencert -initca secrets/ca-csr.json | cfssljson -bare secrets/ca
```

Files created results:

```
ca-key.pem
ca.pem
```

## Client and Server Certificates

### The Admin Client Certificate

Create the `admin` client certificate signing request and generate the `admin` client certificate and private key::

```
cat > secrets/admin-csr.json << EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Seattle",
      "O": "system:masters",
      "OU": "Kubernetes The MaaS Way",
      "ST": "Washington"
    }
  ]
}
EOF
```

```
cfssl gencert \
  -ca=secrets/ca.pem \
  -ca-key=secrets/ca-key.pem \
  -config=secrets/ca-config.json \
  -profile=kubernetes \
  secrets/admin-csr.json | cfssljson -bare secrets/admin
```

Files created results:

```
admin-key.pem
admin.pem
```

### The Kubelet Client Certificates

Generate a certificate and private key for each Kubernetes worker node:

```
for instance in worker-1 worker-2; do
cat > secrets/${instance}-csr.json << EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Seattle",
      "O": "system:nodes",
      "OU": "Kubernetes The MaaS Way",
      "ST": "Washington"
    }
  ]
}
EOF

cfssl gencert \
  -ca=secrets/ca.pem \
  -ca-key=secrets/ca-key.pem \
  -config=secrets/ca-config.json \
  -hostname=${instance} \
  -profile=kubernetes \
  secrets/${instance}-csr.json | cfssljson -bare secrets/${instance}
done
```

Files created results:

```
worker-1-key.pem
worker-1.pem
worker-2-key.pem
worker-2.pem
```

## The Controller Manager Client Certificate

Generate the `kube-controller-manager` client certificate and private key:

```
{
cat > secrets/kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Seattle",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The MaaS Way",
      "ST": "Washington"
    }
  ]
}
EOF

cfssl gencert \
  -ca=secrets/ca.pem \
  -ca-key=secrets/ca-key.pem \
  -config=secrets/ca-config.json \
  -profile=kubernetes \
  secrets/kube-controller-manager-csr.json | cfssljson -bare secrets/kube-controller-manager
}
```

Files created results:

```
kube-controller-manager-key.pem
kube-controller-manager.pem
```

### The kube-proxy Client Certificate

Create the `kube-proxy` client certificate signing request and generate the `kube-proxy` client certificate and private key::

```
cat > secrets/kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Seattle",
      "O": "system:node-proxier",
      "OU": "Kubernetes The MaaS Way",
      "ST": "Washington"
    }
  ]
}
EOF
```

```
cfssl gencert \
  -ca=secrets/ca.pem \
  -ca-key=secrets/ca-key.pem \
  -config=secrets/ca-config.json \
  -profile=kubernetes \
  secrets/kube-proxy-csr.json | cfssljson -bare secrets/kube-proxy
```

Files created results:

```
kube-proxy-key.pem
kube-proxy.pem
```

### The Kubernetes API Server Certificate

Create the Kubernetes API Server certificate signing request and generate the Kubernetes API Server certificate and private key::

> The IP address was assigned to the environment variable CONTROLLER_PUBLIC_ADDRESS in Step 03

```
cat > secrets/kubernetes-csr.json << EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Seattle",
      "O": "Kubernetes",
      "OU": "Kubernetes The MaaS Way",
      "ST": "Washington"
    }
  ]
}
EOF
```

```
cfssl gencert \
  -ca=secrets/ca.pem \
  -ca-key=secrets/ca-key.pem \
  -config=secrets/ca-config.json \
  -hostname=${CONTROLLER_PUBLIC_ADDRESS},127.0.0.1,kubernetes.default \
  -profile=kubernetes \
  secrets/kubernetes-csr.json | cfssljson -bare secrets/kubernetes
```

Files created results:

```
kubernetes-key.pem
kubernetes.pem
```

## Distribute the Client and Server Certificates

Copy the appropriate certificates and private keys to each worker and controller instance:

```
scp secrets/ca.pem secrets/ca-key.pem secrets/kubernetes-key.pem secrets/kubernetes.pem ubuntu@$CONTROLLER_PUBLIC_ADDRESS:~/
```

```
scp secrets/ca.pem secrets/worker-1-key.pem secrets/worker-1.pem ubuntu@$WORKER_1:~/
```

```
scp secrets/ca.pem secrets/worker-2-key.pem secrets/worker-2.pem ubuntu@$WORKER_2:~/
```


Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)