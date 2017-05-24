# Setting up a Certificate Authority and Creating TLS Certificates

In this lab you will setup the necessary PKI infrastructure to secure the Kubernetes components. This lab will leverage CloudFlare's PKI toolkit, [cfssl](https://github.com/cloudflare/cfssl), to bootstrap a Certificate Authority and generate TLS certificates to secure the following Kubernetes components:

* etcd
* kube-apiserver
* kubelet
* kube-proxy

After completing this lab you should have the following TLS keys and certificates:

```
admin.pem
admin-key.pem
ca-key.pem
ca.pem
kubernetes-key.pem
kubernetes.pem
kube-proxy.pem
kube-proxy-key.pem
```

## Install CFSSL

This lab requires the `cfssl` and `cfssljson` binaries. Download them from the [cfssl repository](https://pkg.cfssl.org).

### OS X

```
wget https://pkg.cfssl.org/R1.2/cfssl_darwin-amd64
chmod +x cfssl_darwin-amd64
sudo mv cfssl_darwin-amd64 /usr/local/bin/cfssl
```

```
wget https://pkg.cfssl.org/R1.2/cfssljson_darwin-amd64
chmod +x cfssljson_darwin-amd64
sudo mv cfssljson_darwin-amd64 /usr/local/bin/cfssljson
```

### Linux

```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
chmod +x cfssl_linux-amd64
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
```

```
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssljson_linux-amd64
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
```

## Set up a Certificate Authority

Create a CA configuration file:

```
cat > json/ca-config.json <<EOF
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

Create a CA certificate signing request:


```
cat > json/ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "DE",
      "L": "Berlin",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Berlin"
    }
  ]
}
EOF
```

Generate a CA certificate and private key:

```
cfssl gencert -initca json/ca-csr.json | cfssljson -bare ca
```

Results:

```
ca-key.pem
ca.pem
```

## Generate client and server TLS certificates

In this section we will generate TLS certificates for each Kubernetes component and a client certificate for the admin user.

### Create the Admin client certificate

Create the admin client certificate signing request:

```
cat > json/admin-csr.json <<EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "DE",
      "L": "Berlin",
      "O": "system:masters",
      "OU": "Cluster",
      "ST": "Berlin"
    }
  ]
}
EOF
```

Generate the admin client certificate and private key:

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=json/ca-config.json \
  -profile=kubernetes \
  json/admin-csr.json | cfssljson -bare admin
```

Results:

```
admin-key.pem
admin.pem
```

### Create the kube-proxy client certificate

Create the kube-proxy client certificate signing request:

```
cat > json/kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "DE",
      "L": "Berlin",
      "O": "system:node-proxier",
      "OU": "Cluster",
      "ST": "Berlin"
    }
  ]
}
EOF
```

Generate the kube-proxy client certificate and private key:

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=json/ca-config.json \
  -profile=kubernetes \
  json/kube-proxy-csr.json | cfssljson -bare kube-proxy
```

Results:

```
kube-proxy-key.pem
kube-proxy.pem
```

### Create the kubernetes server certificate

The Kubernetes public IP address will be included in the list of subject alternative names for the Kubernetes server certificate. This will ensure the TLS certificate is valid for remote client access.

```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region us-central1 \
  --format 'value(address)')
```

Create the Kubernetes server certificate signing request:

```
cat > json/kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "hosts": [
    "10.1.232.5",
    "10.1.232.6",
    "10.1.232.7",
    "127.0.0.1",
    "kubernetes.default"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "DE",
      "L": "Berlin",
      "O": "Kubernetes",
      "OU": "Cluster",
      "ST": "Berlin"
    }
  ]
}
EOF
```

Generate the Kubernetes certificate and private key:

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=json/ca-config.json \
  -profile=kubernetes \
  json/kubernetes-csr.json | cfssljson -bare kubernetes
```

Results:

```
kubernetes-key.pem
kubernetes.pem
```

## Distribute the TLS certificates

Set the list of Kubernetes hosts where the certs should be copied to:

The following commands will copy the TLS certificates and keys to each Kubernetes host using the `gcloud compute copy-files` command.

```
for host in oobtest-kubenode-{1..3}-test-01.test.bnotk.net  
do
  scp ca.pem kube-proxy{,-key}.pem root@${host}:~/
done
```

```
for host in oobtest-kubemaster-{1..3}-test-01.test.bnotk.net  
do 
  scp {ca,kubernetes}{,-key}.pem root@${host}:
done
```
