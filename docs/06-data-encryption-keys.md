# Generating the Data Encryption Config and Key

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) cluster data at rest.

In this lab you will generate an encryption key and an [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) suitable for encrypting Kubernetes Secrets.

## The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

### Verification

```
echo $ENCRYPTION_KEY
```

> output

```
NBsr9s3tyz2w0ZWcqOkqTu9PQ2OodZsj+9CFdGDUXtQ=
```

(Some random string of charatcers)

## The Encryption Config File

Create the `encryption-config.yaml` encryption config file:

```
cat > secrets/encryption-config.yaml << EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

Copy the `encryption-config.yaml` encryption config file to the controller instance:

```
scp secrets/encryption-config.yaml ubuntu@$CONTROLLER_PUBLIC_ADDRESS:~/
```

Next: [Bootstrapping the etcd Cluster](07-bootstrapping-etcd.md)