# Kubernetes the Hard Way
Building Highly Available Kubernetes cluster from scratch. 

- [Getting Started](#getting-started)
    - [What Will the Kubernetes Cluster Architecture Look Like?](#what-will-the-kubernetes-cluster-architecture-look-like)
    - [Setting Up Your Cloud Servers](#setting-up-your-cloud-servers)
    - [Client Tools](#client-tools)
- [Provisioning the CA and Generating TLS Certificates](#provisioning-the-ca-and-generating-tls-certificates)
    - [Why Do We Need a CA and TLS Certificates?](#why-do-we-need-a-ca-and-tls-certificates)
    - [Provisioning the Certificate Authority](#provisioning-the-certificate-authority)
    - [Generating Client Certificates](#generating-client-certificates)
    - [Generating the Kubernetes API Server Certificate](#generating-the-kubernetes-api-server-certificate)
    - [Generating the Service Account Key Pair](#generating-the-service-account-key-pair)
    - [Distributing the Certificate Files](#distributing-the-certificate-files)
    - [Creating a Certificate Authority and TLS Certificates for Kubernetes](#creating-a-certificate-authority-and-tls-certificates-for-kubernetes)
- [Generating Kubernetes Configuration Files for Authentication](#generating-kubernetes-configuration-files-for-authentication)
    - [What are Kubeconfigs and Why Do We Need Them?](#what-are-kubeconfigs-and-why-do-we-need-them)
    - [Generating Kubeconfigs for the Cluster](#generating-kubeconfigs-for-the-cluster)
    - [Distributing the Kubeconfig Files](#distributing-the-kubeconfig-files)
    - [Generating Kubeconfigs for a New Kubernetes Cluster](#generating-kubeconfigs-for-a-new-kubernetes-cluster)
- [Generating the Data Encryption Config and Key](#generating-the-data-encryption-config-and-key)
    - [What is the Data Encryption Config in Kubernetes?](#what-is-the-data-encryption-config-in-kubernetes)
    - [Generating the Data Encryption Config](#generating-the-data-encryption-config)
    - [Generating a Data Encryption Config for Kubernetes](#generating-a-data-encryption-config-for-kubernetes)
- [Bootstrapping the etcd Cluster](#bootstrapping-the-etcd-cluster)
    - [What is etcd?](#what-is-etcd)
    - [Creating the etcd Cluster](#creating-the-etcd-cluster)
    - [Bootstrapping an etcd Cluster for Kubernetes](#bootstrapping-an-etcd-cluster-for-kubernetes)
- [Bootstrapping the Kubernetes Control Plane](#bootstrapping-the-kubernetes-control-plane)
    - [What is the Kubernetes Control Plane?](#what-is-the-kubernetes-control-plane)
    - [Control Plane Architecture Overview](#control-plane-architecture-overview)
    - [Installing Kubernetes Control Plane Binaries](#installing-kubernetes-control-plane-binaries)
    - [Setting up the Kubernetes API Server](#setting-up-the-kubernetes-api-server)
    - [Setting up the Kubernetes Controller Manager](#setting-up-the-kubernetes-controller-manager)
    - [Setting up the Kubernetes Scheduler](#setting-up-the-kubernetes-scheduler)
    - [Enable HTTP Health Checks](#enable-http-health-checks)
    - [Set up RBAC for Kubelet Authorization](#set-up-rbac-for-kubelet-authorization)
    - [Setting up a Kube API Frontend Load Balancer](#setting-up-a-kube-api-frontend-load-balancer)
    - [Bootstrapping a Kubernetes Control Plane](#bootstrapping-a-kubernetes-control-plane)
    - [Setting Up a Frontend Load Balancer for the Kubernetes API](#setting-up-a-frontend-load-balancer-for-the-kubernetes-api)
- [Bootstrapping the Kubernetes Worker Nodes](#bootstrapping-the-kubernetes-worker-nodes)
    - [What are the Kubernetes Worker Nodes?](#what-are-the-kubernetes-worker-nodes)
    - [Worker Node Architecture Overview](#worker-node-architecture-overview)
    - [Installing Worker Node Binaries](#installing-worker-node-binaries)
    - [Configuring Containerd](#configuring-containerd)
    - [Configuring Kube-Proxy](#configuring-kube-proxy)
    - [Bootstrapping Kubernetes Worker Nodes](#bootstrapping-kubernetes-worker-nodes)
- [Configuring kubectl for Remote Access](#configuring-kubectl-for-remote-access)
    - [Kubernetes Remote Access and kubectl](#kubernetes-remote-access-and-kubectl)
    - [Configuring Kubectl for Remote Access](#configuring-kubectl-for-remote-access)
    - [Configuring Kubectl to Access a Remote Cluster](#configuring-kubectl-to-access-a-remote-cluster)
- [Networking](#networking)
    - [The Kubernetes Networking Model](#the-kubernetes-networking-model)
    - [Cluster Network Architecture](#cluster-network-architecture)
    - [Installing Weave Net](#installing-weave-net)
    - [Setting Up Kubernetes Networking with Weave Net](#setting-up-kubernetes-networking-with-weave-net)
- [Deploying the DNS Cluster Add-on](#deploying-the-dns-cluster-add-on)
    - [DNS In a Kubernetes Pod Network](#dns-in-a-kubernetes-pod-network)
    - [Deploying kube-dns in a Kubernetes Cluster](#deploying-kube-dns-in-a-kubernetes-cluster)
- [Smoke Test](#smoke-test)
    - [Smoke Testing the Cluster](#smoke-testing-the-cluster)
    - [Smoke Testing Data Encryption](#smoke-testing-data-encryption)
    - [Smoke Testing Deployments](#smoke-testing-deployments)

## Getting Started 
### What Will the Kubernetes Cluster Architecture Look Like?
Structure going to be build during the course, according to the Kubernetes the Hard Way documentation.
  
![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img1.png)


### Setting Up Your Cloud Servers
Let's start from creating machines on our Playground. We will have 2 masters, 2 workers and 1 load balancer. 

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img2.png)

### Client Tools
In order to proceed with *Kubernetes the Hard Way*, there are some client tools that you need to install on your local workstation. These include cfssl and kubectl. This lesson introduces these tools and guides you through the process of installing them. After completing this lesson, you should have cfssl and kubectl installed correctly on your workstation.
  
You can find more information on how to install these tools, as well as instructions for OS X/Linux, here: https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-client-tools.md

**Commands used in the demo to install the client tools in a Linux environment:**
  
cfssl:
```
wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
cfssl version
```
  
If you want to work on an i386 machine, use these commands to install cfssl instead:
```
wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-386 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-386
chmod +x cfssl_linux-386 cfssljson_linux-386
sudo mv cfssl_linux-386 /usr/local/bin/cfssl
sudo mv cfssljson_linux-386 /usr/local/bin/cfssljson
cfssl version
```
  
kubectl:
```
wget https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

## Provisioning the CA and Generating TLS Certificates
### Why Do We Need a CA and TLS Certificates?
In this section of the course, we will be creating the certificate authority and various TLS certificates that we will need later to set up the Kubernetes cluster. This lesson provides some background on why we need these certificates and what they will be used for in Kubernetes. After completing this lesson, you will have a basic understanding of the context for the tasks that will be performed in this section. These tasks will provide you with a better understanding of what you will be doing as you provision the certificate authority and generate the TLS certificates.

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img3.png)

### Provisioning the Certificate Authority
In order to generate the certificates needed by Kubernetes, you must first provision a certificate authority. This lesson will guide you through the process of provisioning a new certificate authority for your Kubernetes cluster. After completing this lesson, you should have a certificate authority, which consists of two files: `ca-key.pem` and `ca.pem`.
  
Here are the commands used in the demo:
```
cd ~/
mkdir kthw
cd kthw/
```
  
UPDATE: cfssljson and cfssl will need to be installed. To install, complete the following commands:
```
sudo curl -s -L -o /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
sudo curl -s -L -o /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
sudo curl -s -L -o /bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
sudo chmod +x /bin/cfssl*
```
  
Use this command to generate the certificate authority. Include the opening and closing curly braces to run this entire block as a single command.
```
{

cat > ca-config.json << EOF
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

cat > ca-csr.json << EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

### Generating Client Certificates
Now that you have provisioned a certificate authority for the Kubernetes cluster, you are ready to begin generating certificates. The first set of certificates you will need to generate consists of the client certificates used by various Kubernetes components. In this lesson, we will generate the following client certificates: `admin`, `kubelet` (one for each worker node), `kube-controller-manager`, `kube-proxy`, and `kube-scheduler`. After completing this lesson, you will have the client certificate files which you will need later to set up the cluster.
  
Here are the commands used in the demo. The command blocks surrounded by curly braces can be entered as a single command:
```
cd ~/kthw
```
  
Admin Client certificate:
```
{

cat > admin-csr.json << EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}
```
  
Kubelet Client certificates. Be sure to enter your actual cloud server values for all four of the variables at the top:
```
WORKER0_HOST=<Public hostname of your first worker node cloud server>
WORKER0_IP=<Private IP of your first worker node cloud server>
WORKER1_HOST=<Public hostname of your second worker node cloud server>
WORKER1_IP=<Private IP of your second worker node cloud server>

{
cat > ${WORKER0_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER0_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER0_IP},${WORKER0_HOST} \
  -profile=kubernetes \
  ${WORKER0_HOST}-csr.json | cfssljson -bare ${WORKER0_HOST}

cat > ${WORKER1_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER1_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER1_IP},${WORKER1_HOST} \
  -profile=kubernetes \
  ${WORKER1_HOST}-csr.json | cfssljson -bare ${WORKER1_HOST}

}
```
  
Controller Manager Client certificate:
```
{

cat > kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```
  
Kube Proxy Client certificate:
```
{

cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```
  
Kube Scheduler Client Certificate:
```
{

cat > kube-scheduler-csr.json << EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```
  
As a result we will have ca.pem (Public key) and ca-key.pem (Private key) certificates

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img4.png)


### Generating the Kubernetes API Server Certificate
We have generated all of the the client certificates our Kubernetes cluster will need, but we also need a server certificate for the Kubernetes API. In this lesson, we will generate one, signed with all of the hostnames and IPs that may be used later in order to access the Kubernetes API. After completing this lesson, you will have a Kubernetes API server certificate in the form of two files called `kubernetes-key.pem` and `kubernetes.pem`.
  
Here are the commands used in the demo. Be sure to replace all the placeholder values in **CERT_HOSTNAME** with their real values from your cloud servers:

```
cd ~/kthw

CERT_HOSTNAME=10.32.0.1,<controller node 1 Private IP>,<controller node 1 hostname>,
<controller node 2 Private IP>,<controller node 2 hostname>,<API load balancer
Private IP>,<API load balancer hostname>,127.0.0.1,localhost,kubernetes.default

{

cat > kubernetes-csr.json << EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${CERT_HOSTNAME} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
```

### Generating the Service Account Key Pair
Kubernetes provides the ability for service accounts to authenticate using tokens. It uses a key-pair to provide signatures for those tokens. In this lesson, we will generate a certificate that will be used as that key-pair. After completing this lesson, you will have a certificate ready to be used as a service account key-pair in the form of two files: `service-account-key.pem` and `service-account.pem`.
  
Here are the commands used in the demo:
```
cd ~/kthw

{

cat > service-account-csr.json << EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}
```

### Distributing the Certificate Files
Now that all of the necessary certificates have been generated, we need to move the files onto the appropriate servers. In this lesson, we will copy the necessary certificate files to each of our cloud servers. After completing this lesson, your controller and worker nodes should each have the certificate files which they need.
  
Here are the commands used in the demo. Be sure to replace the placeholders with the actual values from from your cloud servers.
  
Move certificate files to the worker nodes:
```
scp ca.pem <worker 1 hostname>-key.pem <worker 1 hostname>.pem user@<worker 1 public IP>:~/
scp ca.pem <worker 2 hostname>-key.pem <worker 2 hostname>.pem user@<worker 2 public IP>:~/
```
  
Move certificate files to the controller nodes:
```
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem user@<controller 1 public IP>:~/
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem user@<controller 2 public IP>:~/
```

### Creating a Certificate Authority and TLS Certificates for Kubernetes
##### Additional Information and Resources
Your team is working on setting up a Kubernetes cluster with two controllers and two worker nodes. To enable all the components of Kubernetes to securely authenticate with each other, your team needs to provision a certificate authority and generate several certificates using that authority. Your task is to create the certificate authority and the necessary certificates.
  
You will need to log into the learning activity server using the `Workspace Public IP`. This server already has cfssl installed, so there is no need to install it.
  
In order to accomplish this, you need to:

- Provision the certificate authority (CA)
- Generate the necessary Kubernetes client certs, as well as kubelet client certs for
two worker nodes.
- Generate the Kubernetes API server certificate.
- Generate a Kubernetes service account key pair.
  
Click the icon next to each task below for more information on how to complete each task. You can also check out the solution video for a detailed walkthrough.
  
Here is the cluster architecture for which you will need to generate certificates. Note that these are not real servers, just values that we will use for the purposes of this activity.
  
- Controllers:
  - Hostname: controller0.mylabserver.com, IP: 172.34.0.0
  - Hostname: controller1.mylabserver.com, IP: 172.34.0.1

- Workers:
  - Hostname: worker0.mylabserver.com, IP: 172.34.1.0
  - Hostname: worker1.mylabserver.com, IP: 172.34.1.1

- Kubernetes API Load Balancer:
  - Hostname: kubernetes.mylabserver.com, IP: 172.34.2.0

**Provision the certificate authority (CA).**
- You can provision the certificate authority like so:
```
{

cat > ca-config.json << EOF
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

cat > ca-csr.json << EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

**Generate the necessary Kubernetes client certs, as well as kubelet client certs for two worker nodes.**
- Use these commands to generate the client certs.
- Admin client cert:
```
{

cat > admin-csr.json << EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}
```

- Kubelet client certs:
```
{
cat > worker0.mylabserver.com-csr.json << EOF
{
  "CN": "system:node:worker0.mylabserver.com",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=172.34.1.0,worker0.mylabserver.com \
  -profile=kubernetes \
  worker0.mylabserver.com-csr.json | cfssljson -bare worker0.mylabserver.com

cat > worker1.mylabserver.com-csr.json << EOF
{
  "CN": "system:node:worker1.mylabserver.com",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=172.34.1.1,worker1.mylabserver.com \
  -profile=kubernetes \
  worker1.mylabserver.com-csr.json | cfssljson -bare worker1.mylabserver.com

}
```

- Kube Controller Manager client cert:
```
{

cat > kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```

- Kube Proxy client cert:
```


cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```

- Kube Scheduler client cert:
```
{

cat > kube-scheduler-csr.json << EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```

**Generate the Kubernetes API server certificate.**
- You can generate the Kubernetes API server certificate like so:
```
{

cat > kubernetes-csr.json << EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,172.34.0.0,controller0.mylabserver.com,172.34.0.1,controller1.mylabserver.com,172.34.2.0,kubernetes.mylabserver.com,127.0.0.1,localhost,kubernetes.default \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
```

**Generate a Kubernetes service account key pair.**
- To generate the service account key pair, do the following:
```
{

cat > service-account-csr.json << EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}
```

## Generating Kubernetes Configuration Files for Authentication
### What are Kubeconfigs and Why Do We Need Them?
In this section, we will be generating kubeconfigs which will be used later to set up a Kubernetes cluster. Before proceeding, however, it is a good idea to have some basic understanding of what kubeconfigs are, and what purpose they serve. This lesson introduces you to kubeconfigs and provides some background on what they do and why they are needed. After completing this lesson, you will have an idea of how kubeconfigs are used, which will prepare you for the task of creating them.
  
You can find more information on kubeconfigs in the official Kubernetes documentation: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

### Generating Kubeconfigs for the Cluster
The next step in building a Kubernetes cluster the hard way is to generate kubeconfigs which will be used by the various services that will make up the cluster. In this lesson, we will generate these kubeconfigs. After completing this lesson, you should have a set of kubeconfigs which you will need later in order to configure the Kubernetes cluster.
  

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img5.png)

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img6.png)

- Here are the commands used in the demo. Be sure to replace the placeholders with actual values from your cloud servers.
  
- Create an environment variable to store the address of the Kubernetes API, and set it to the private IP of your load balancer cloud server:
```
KUBERNETES_ADDRESS=<load balancer private ip>
```

- Generate a kubelet kubeconfig for each worker node:
```
for instance in <worker 1 hostname> <worker 2 hostname>; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

- Generate a kube-proxy kubeconfig:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

- Generate a kube-controller-manager kubeconfig:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

- Generate a kube-scheduler kubeconfig:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

- Generate an admin kubeconfig:

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

### Distributing the Kubeconfig Files
Now that we have generated the kubeconfig files that we will need in order to configure our Kubernetes cluster, we need to make sure that each cloud server has a copy of the kubeconfig files that it will need. In this lesson, we will distribute the kubeconfig files to each of the worker and controller nodes so that they will be in place for future lessons. After completing this lesson, each of your worker and controller nodes should have a copy of the kubeconfig files it needs.

- Here are the commands used in the demo. Be sure to replace the placeholders with the actual values from your cloud servers.

- Move kubeconfig files to the worker nodes:
```
scp <worker 1 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 1 public IP>:~/
scp <worker 2 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 2 public IP>:~/
```

- Move kubeconfig files to the controller nodes:
```
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 1 public IP>:~/
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 2 public IP>:~/
```

### Generating Kubeconfigs for a New Kubernetes Cluster
##### Additional Information and Resources
Your team is setting up a new Kubernetes cluster with two controllers and two worker nodes. The team has already created a set of client certificates to allow different components of the cluster to authenticate, but they need a set of kubeconfig files to be created using these certificates. Your task is to generate the kubeconfig files that will be used to set up the Kubernetes cluster.
  
The following kubeconfig files need to be created:

- Kubelet (one kubeconfig for each worker node)
- Kube-proxy
- Kube-controller-manager
- Kube-scheduler
- Admin
Here is the cluster architecture. Note that these are not real servers, just values that we will use for the purposes of this activity.

- Controllers:
  - Hostname: controller0.mylabserver.com, IP: 172.34.0.0
  - Hostname: controller1.mylabserver.com, IP: 172.34.0.1

- Workers:
  - Hostname: worker0.mylabserver.com, IP: 172.34.1.0
  - Hostname: worker1.mylabserver.com, IP: 172.34.1.1

- Kubernetes API Load Balancer:
  - Hostname: kubernetes.mylabserver.com, IP: 172.34.2.0

- Additional Notes:
  - Client certificates have already been created. They can be found in `/home/cloud_user` on the workspace server.
  - The `kubelet` and `kube-proxy` services will access the Kubernetes API through the Kubernetes API load balancer. All other services can access it locally at https://127.0.0.1:6443.

**Generate kubelet kubeconfigs for each worker node.**
- To complete this task, generate a `kubelet` kubeconfig for each worker node. You can do so like this:
```
KUBERNETES_PUBLIC_ADDRESS=172.34.2.0

for instance in worker0.mylabserver.com worker1.mylabserver.com; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

**Generate a kube-proxy kubeconfig.**
- To complete this task, generate a kubeconfig for `kube-proxy`. You can do so like this:
```
KUBERNETES_PUBLIC_ADDRESS=172.34.2.0

{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

**Generate a kube-controller-manager kubeconfig.**
- To complete this task, generate a kubeconfig for `kube-controller-manager`. You can do so like this:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

**Generate a kube-scheduler kubeconfig.**
- To complete this task, generate a kubeconfig for `kube-scheduler`. You can do so like this:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

**Generate an admin kubeconfig.**
To complete this task, generate a kubeconfig for the `admin` user. You can do so like this:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

## Generating the Data Encryption Config and Key
### What is the Data Encryption Config in Kubernetes?
One important security practice is to ensure that sensitive data is never stored in plain text. Kubernetes offers the ability to encrypt sensitive data when it is stored. However, in order to use this feature it is necessary to provide Kubernetes with a data encrpytion config containing an encryption key. This lesson briefly discusses what the data encrpytion config is and why it is needed. This will provide you with some background knowledge as you proceed to create a data encrpytion config for your cluster. After completing this lesson, you will have a basic understanding of what a data encrpytion config is and what it is used for.
  
You can find more information on data encrpytion in Kubernetes in the official docs: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img7.png)


### Generating the Data Encryption Config
In order to make use of Kubernetes' ability to encrypt sensitive data at rest, you need to provide Kubernetes with an encrpytion key using a data encrpyiton config file. This lesson walks you through the process of creating a encryption key and storing it in the necessary file, as well as showing how to copy that file to your Kubernetes controllers. After completing this lesson, you should have a valid Kubernetes data encyption config file, and there should be a copy of that file on each of your Kubernetes controller servers.
  
Here are the commands used in the demo.
  
- Generate the Kubernetes Data encrpytion config file containing the encrpytion key:
```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

cat > encryption-config.yaml << EOF
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

- Copy the file to both controller servers:
```
scp encryption-config.yaml user@<controller 1 public ip>:~/
scp encryption-config.yaml user@<controller 2 public ip>:~/
```

### Generating a Data Encryption Config for Kubernetes
##### Additional Information and Resources
Your team is working on setting up a Kubernetes cluster with two controllers and two worker nodes. In order to ensure that the cluster is configured securely, the team wants to enable the feature that allows Kubernetes to encrypt sensitive data at rest. In order to accomplish this, the team needs a Kubernetes data encryption config file containing an encryption key. Your task is to generate an encryption key and create this file, then copy the file to the two Kubernetes master servers.
  
**Generate an encryption key and include it in a Kubernetes data encryption config file.**
- To accomplish this task, do the following on the workspace server:
```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

cat > encryption-config.yaml << EOF
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

**Copy the file to the Kubernetes controller servers.**
- Copy `encryption-config.yaml` to each Kubernetes controller by running these commands from the workspace server. Be sure to replace the placeholders with the actual IP addresses of the controller servers.
```
scp encryption-config.yaml cloud_user@<controller 1 public ip>:~/
scp encryption-config.yaml cloud_user@<controller 2 public ip>:~/
```

## Bootstrapping the etcd Cluster
### What is etcd?
In this section, we will be setting up an etcd cluster, which is a necessary component of our Kubernetes cluster. In order to fully understand this process, however, it is a good idea to have some idea of what etcd and the role it plays in Kubernetes. This lesson introduces you to etcd and discusses how Kubernetes uses it. After completing this lesson, you will have some background knowledge to prepare you for the task of installing and configuring etcd.

- You can find more information about etcd in the following locations:
  - https://coreos.com/etcd/
  - https://github.com/coreos/etcd (this GitHub repository also containes the etcd source code)

- Check out the Kubernetes documentation for more information on managing etcd in the context of a Kubernetes cluster:
  - https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img8.png)

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img9.png)

### Creating the etcd Cluster
Before you can stand up controllers for a Kubernetes cluster, you must first build an etcd cluster across your Kubernetes control nodes. This lesson provides a demonstration of how to set up an etcd cluster in preparation for bootstrapping Kubernetes. After completing this lesson, you should have a working etcd cluster that consists of your Kubernetes control nodes.

- Here are the commands used in the demo (note that these have to be run on both controller servers, with a few differences between them):
```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"
tar -xvf etcd-v3.3.5-linux-amd64.tar.gz
sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/
sudo mkdir -p /etc/etcd /var/lib/etcd
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

- Set up the following environment variables. Be sure you replace all of the `<placeholder values>` with their corresponding real values:
```
ETCD_NAME=<cloud server hostname>
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
INITIAL_CLUSTER=<controller 1 hostname>=https://<controller 1 private ip>:2380,<controller 2 hostname>=https://<controller 2 private ip>:2380
```

- Create the `systemd` unit file for etcd using this command. Note that this command uses the environment variables that were set earlier:
```
cat << EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${INITIAL_CLUSTER} \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

- Start and enable the etcd service:
```
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

- You can verify that the etcd service started up successfully like so:
```
sudo systemctl status etcd
```

- Use this command to verify that etcd is working correctly. The output should list your two etcd nodes:
```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

### Bootstrapping an etcd Cluster for Kubernetes
##### Additional Information and Resources
Your team is working on setting up a new Kubernetes cluster. Because etcd is one of the necessary components of Kubernetes, the team needs an etcd cluster configured to run across all of the servers that will become the Kubernetes control nodes. You have been given the task of setting up an etcd cluster that will be used to support Kubernetes.
  
You can verify that your etcd cluster is working like this:
```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

##### Install the etcd binary on both control nodes.
- Do the following on both Kubernetes control nodes:
```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"
tar -xvf etcd-v3.3.5-linux-amd64.tar.gz
sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/
```

##### Configure and start the etcd service on both control nodes.
- Do the following on both Kubernetes control nodes:
```
sudo mkdir -p /etc/etcd /var/lib/etcd
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

- Set the `ETCD_NAME` variable. Be sure to replace the placeholder in this command with `controller-0` or `controller-1`, as appropriate for each server.
```
ETCD_NAME=<controller-0 or controller-1>
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
CONTROLLER_0_INTERNAL_IP=<controller 0 private ip>
CONTROLLER_1_INTERNAL_IP=<controller 1 private ip>
```

- Create the etcd systemd unit file with the following command. Be sure to replace the placeholders in `--name` and `--initial-cluster` with real values.
```
cat << EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://${CONTROLLER_0_INTERNAL_IP}:2380,controller-1=https://${CONTROLLER_1_INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

- Start and enable the etcd service:
```
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

## Bootstrapping the Kubernetes Control Plane
### What is the Kubernetes Control Plane?
In this section, we will be setting up a distributed Kubernetes control plane. This lesson introduces the control plane and provides a brief overview of the various components that you will be working with. After completing this lesson, you should have an idea of what the Kubernetes control plane does, and you will have a basic understanding of the components that you will be installing and configuring throughout this section of the course.

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img10.png)

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img11.png)

- You can find more information on the Kubernetes control plane in the official docs: https://kubernetes.io/docs/concepts/overview/components/#master-components


### Control Plane Architecture Overview
This lesson provides a brief overview of the architectural end-state of this section of the course. After completing this lesson, you should have an understanding of what this section is seeking to accomplish, and what your Kubernetes cluster will look like after this section is completed. You will then be ready to begin the process of actually building out your own Kubernetes control plane!

- In this module we're going to configure Controller 1 and Controller 2 nodes. 
  - etc - already installed
  - kube-apiserver 
  - small nginx server with healthz monitoring
  - kube-controller-manager
  - kube-scheduler
  - kube api load balancer

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img12.png)

### Installing Kubernetes Control Plane Binaries
The first step in bootstrapping a new Kubernetes control plane is to install the necessary binaries on the controller servers. We will walk through the process of downloading and installing the binaries on both Kubernetes controllers. This will prepare your environment for the lessons that follow, in which we will configure these binaries to run as `systemd` services.
  
You can install the control plane binaries on each control node like this:
```
sudo mkdir -p /etc/kubernetes/config

wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl"

chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl

sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```

### Setting up the Kubernetes API Server
The Kubernetes API server provides the primary interface for the Kubernetes control plane and the cluster as a whole. When you interact with Kubernetes, you are nearly always doing it through the Kubernetes API server. This lesson will guide you through the process of configuring the kube-apiserver service on your two Kubernetes control nodes. After completing this lesson, you should have a `systemd` unit set up to run kube-apiserver as a service on each Kubernetes control node.

- You can configure the Kubernetes API server like so:
```
sudo mkdir -p /var/lib/kubernetes/

sudo cp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
  service-account-key.pem service-account.pem \
  encryption-config.yaml /var/lib/kubernetes/
```

- Set some environment variables that will be used to create the `systemd` unit file. Make sure you replace the placeholders with their actual values:
```
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
CONTROLLER0_IP=<private ip of controller 0>
CONTROLLER1_IP=<private ip of controller 1>
```

- Generate the kube-apiserver unit file for `systemd`:
```
cat << EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://$CONTROLLER0_IP:2379,https://$CONTROLLER1_IP:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2 \\
  --kubelet-preferred-address-types=InternalIP,InternalDNS,Hostname,ExternalIP,ExternalDNS
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Setting up the Kubernetes Controller Manager
Now that we have set up `kube-apiserver`, we are ready to configure kube-controller-manager. This lesson walks you through the process of configuring a `systemd` service for the Kubernetes Controller Manager. After completing this lesson, you should have the kubeconfig and `systemd` unit file set up and ready to run the kube-controller-manager service on both of your control nodes.

- You can configure the Kubernetes Controller Manager like so:
```
sudo cp kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

- Generate the kube-controller-manager `systemd` unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Setting up the Kubernetes Scheduler
Now we are ready to set up the Kubernetes scheduler. This lesson will walk you through the process of configuring the kube-scheduler `systemd` service. Since this is the last of the three control plane services that need to be set up in this section, this lesson also guides you through through enabling and starting all three services on both control nodes. Finally, this lesson shows you how to verify that your Kubernetes controllers are healthy and working so far. After completing this lesson, you will have a basic, working, Kuberneets control plane distributed across your two control nodes.

- You can configure the Kubernetes Sheduler like this.
- Copy `kube-scheduler.kubeconfig` into the proper location:
```
sudo cp kube-scheduler.kubeconfig /var/lib/kubernetes/
```

- Generate the kube-scheduler yaml config file.
```
cat << EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

- Create the kube-scheduler `systemd` unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

- Start and enable all of the services:
```
sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

- It's a good idea to verify that everything is working correctly so far: Make sure all the services are `active (running)`:
```
sudo systemctl status kube-apiserver kube-controller-manager kube-scheduler
```

- Use kubectl to check componentstatuses. `admin.kubeconfig` used here for authenticating with Kubernetes API:
```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

- You should get output that looks like this:
```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

### Enable HTTP Health Checks
Part of Kelsey Hightower's original Kubernetes the Hard Way guide involves setting up an nginx proxy on each controller to provide access to the Kubernetes API /healthz endpoint over http. This lesson explains the reasoning behind the inclusion of that step and guides you through the process of implementing the http /healthz proxy.

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img13.png)

- You can set up a basic nginx proxy for the healthz endpoint by first installing nginx
```
sudo apt-get install -y nginx
```

- Create an nginx configuration for the health check proxy:
```
cat > kubernetes.default.svc.cluster.local << EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
```

- Set up the proxy configuration so that it is loaded by nginx:
```
sudo mv kubernetes.default.svc.cluster.local /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
sudo systemctl restart nginx
sudo systemctl enable nginx
```

- You can verify that everything is working like so:
```
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
```

- Outpus should be:
```
HTTP/1.1 200 OK
Server: nginx/1.10.3 (Ubuntu)
Date: Wed, 05 Feb 2020 19:19:20 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 2
Connection: keep-alive
```

### Set up RBAC for Kubelet Authorization
One of the necessary steps in setting up a new Kubernetes cluster from scratch is to assign permissions that allow the Kubernetes API to access various functionality within the worker kubelets. This lesson guides you through the process of creating a ClusterRole and binding it to the kubernetes user so that those permissions will be in place. After completing this lesson, your cluster will have the necessary role-based access control configuration to allow the cluster's API to access kubelet functionality such as logs and metrics.

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img14.png)

- You can configure RBAC for kubelet authorization with these commands. Note that these commands only need to be run on one control node.

- Create a role with the necessary permissions:
```
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

- Bind the role to the kubernetes user:
```
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

### Setting up a Kube API Frontend Load Balancer
In order to achieve redundancy for your Kubernetes cluster, you will need to load balance usage of the Kubernetes API across multiple control nodes. In this lesson, you will learn how to create a simple nginx server to perform this balancing. After completing this lesson, you will be able to interact with both control nodes of your kubernetes cluster using the nginx load balancer.

- Here are the commands you can use to set up the nginx load balancer. Run these on the server that you have designated as your load balancer server:

```
sudo apt-get install -y nginx
sudo systemctl enable nginx
sudo mkdir -p /etc/nginx/tcpconf.d
sudo vi /etc/nginx/nginx.conf
```

- Add the following to the end of nginx.conf:
```
include /etc/nginx/tcpconf.d/*;
```

- Set up some environment variables for the lead balancer config file:
```
CONTROLLER0_IP=<controller 0 private ip>
CONTROLLER1_IP=<controller 1 private ip>
```

- Create the load balancer nginx config file:
```
cat << EOF | sudo tee /etc/nginx/tcpconf.d/kubernetes.conf
stream {
    upstream kubernetes {
        server $CONTROLLER0_IP:6443;
        server $CONTROLLER1_IP:6443;
    }

    server {
        listen 6443;
        listen 443;
        proxy_pass kubernetes;
    }
}
EOF
```

- Reload the nginx configuration:
```
sudo nginx -s reload
```

- You can verify that the load balancer is working like so:
```
curl -k https://localhost:6443/version
```

- You should get back some json containing version information for your Kubernetes cluster.
```
{
  "major": "1",
  "minor": "10",
  "gitVersion": "v1.10.2",
  "gitCommit": "81753b10df112992bf51bbc2c2f85208aad78335",
  "gitTreeState": "clean",
  "buildDate": "2018-04-27T09:10:24Z",
  "goVersion": "go1.9.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

### Bootstrapping a Kubernetes Control Plane
##### Additional Information and Resources
Your team is working on setting up a new Kubernetes cluster. The necessary certificates and kubeconfigs have been provisioned, and the etcd cluster has been built. Two servers have been created that will become the Kubernetes controller nodes. You have been given the task of building out the Kubernetes control plane by installing and configuring the necessary Kubernetes services: kube-apiserver, kube-controller-manager, and kube-scheduler. You will also need to install kubectl.

##### Download and install the binaries.
- To accomplish this, you will need to download and install the binaries for kube-apiserver, kube-controller-manager, kube-scheduler, and kubectl. You can do so like this:
```
sudo mkdir -p /etc/kubernetes/config

wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl"

chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl

sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```

##### Configure the kube-apiserver service.
- To configure the kube-apiserver service, do the following:
```
sudo mkdir -p /var/lib/kubernetes/

sudo cp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
  service-account-key.pem service-account.pem \
  encryption-config.yaml /var/lib/kubernetes/

INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
```

- Set environment variables to contain the private IPs of both controller servers. Be sure to replace the placeholders with the actual private IPs:
```
ETCD_SERVER_0=<controller 0 private ip>

ETCD_SERVER_1=<controller 1 private ip>
```

- Create the systemd unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://${ETCD_SERVER_0}:2379,https://${ETCD_SERVER_1}:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2 \\
  --kubelet-preferred-address-types=InternalIP,InternalDNS,Hostname,ExternalIP,ExternalDNS
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

##### Configure the kube-controller-manager service.
- To configure the kube-controller-manager `systemd` service, do the following:
```
sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/

cat << EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

##### Configure the kube-scheduler service.
- To configure the kube-scheduler `systemd` service, do the following:
```
sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/

cat << EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF

cat << EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

##### Successfully start all of the services.
- You can start the Kubernetes control plane services like this:
```
sudo systemctl daemon-reload

sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler

sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

- You can verify that everything is working with this command:
```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

- The output should look something like this:
```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

##### Enable HTTP health checks.
- You can enable HTTP health checks like this:
```
sudo apt-get install -y nginx

cat > kubernetes.default.svc.cluster.local << EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF

sudo mv kubernetes.default.svc.cluster.local \
  /etc/nginx/sites-available/kubernetes.default.svc.cluster.local

sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/

sudo systemctl restart nginx

sudo systemctl enable nginx
```

- You can verify that the HTTP health checks are working on each control node like this:
```
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
```

- This should return a `200 OK` status code.

##### Set up RBAC for kubelet authorization.
- You can set up role-based access control for kubelets like this. Note that you only need to do this step on **one** of the controller nodes:
```
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF

cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

### Setting Up a Frontend Load Balancer for the Kubernetes API
##### Additional Information and Resources
Your team is working on setting up a new Kubernetes cluster. Two Kubernetes controllers have been configured, but the team wants to use a load balancer to manage traffic to the Kubernetes API. You have been given the task of setting up an Nginx load balancer that will balance traffic across the Kubernetes API services running on the two Kubernetes controllers.

##### Install Nginx on the load balancer server.
- You can install Nginx like this:
```
sudo apt-get install -y nginx
sudo systemctl enable nginx
```

##### Configure Nginx to balance Kubernetes API traffic across the two controllers.
- Do the following to configure the Nginx load balancer:
```
sudo mkdir -p /etc/nginx/tcpconf.d
sudo vi /etc/nginx/nginx.conf
```

- Add the following configuration at the bottom of `nginx.conf`:
```
include /etc/nginx/tcpconf.d/*;
```

- Create a config file to configure Kubernetes API load balancing:
```
cat << EOF | sudo tee /etc/nginx/tcpconf.d/kubernetes.conf
stream {
    upstream kubernetes {
        server <controller 0 private ip>:6443;
        server <controller 1 private ip>:6443;
    }

    server {
        listen 6443;
        listen 443;
        proxy_pass kubernetes;
    }
}
EOF
```

- Reload the Nginx configuration:
```
sudo nginx -s reload
```

- You can verify that everything is working by making a request to the Kubernetes API through the load balancer:
```
curl -k https://localhost:6443/version
```

- This request should return some Kubernetes version data.
```
{
  "major": "1",
  "minor": "10",
  "gitVersion": "v1.10.2",
  "gitCommit": "81753b10df112992bf51bbc2c2f85208aad78335",
  "gitTreeState": "clean",
  "buildDate": "2018-04-27T09:10:24Z",
  "goVersion": "go1.9.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

## Bootstrapping the Kubernetes Worker Nodes
### What are the Kubernetes Worker Nodes?
Now that we have set up the Kubernetes control plane, we are ready to begin setting up worker nodes. This lesson introduces Kubernetes worker nodes and describes the various components that we will need for installing and configuring them. After completing this lesson, you will have a basic understanding of what worker nodes do and the components that are used to create a worker node.

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img15.png)

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img16.png)

- You can find more information on Kubernetes the worker nodes in the official documentation:
  - https://kubernetes.io/docs/concepts/architecture/
  - https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/architecture.md#the-kubernetes-node

### Worker Node Architecture Overview
This lesson provides a brief overview of the end-state architecture, focusing on the two worker nodes. It discusses how the worker nodes fit into the overall architecture. After completing this lesson, you will have a reference point to help you understand what is being implemented in the following lessons as you continue seting up your worker nodes.

- We're going to configure `Worker 1` and `Worker 2`.
- `kubelet` and `kube-proxy` are going to communicate with controller nodes through API load balancer and kube-apiserver. 

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img17.png)

### Installing Worker Node Binaries
We are now ready to begin the process of setting up our worker nodes. The first step is to download and install the binary file which we will later use to configure our worker nodes services. In this lesson, we will be downloading and installing the binaries for containerd, kubectl, kubelet, and kube-proxy, as well as other software that they depend on. After completing this lesson, you should have these binaries downloaded and all of the files moved into the correct locations in preparation for configuring the worker node services.

- You can install the worker binaries like so. Run these commands on both worker nodes:
```
sudo apt-get -y install socat conntrack ipset

wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.0/crictl-v1.0.0-beta.0-linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-the-hard-way/runsc \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.1.0/containerd-1.1.0.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubelet

sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes

chmod +x kubectl kube-proxy kubelet runc.amd64 runsc

sudo mv runc.amd64 runc

sudo mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/

sudo tar -xvf crictl-v1.0.0-beta.0-linux-amd64.tar.gz -C /usr/local/bin/

sudo tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/

sudo tar -xvf containerd-1.1.0.linux-amd64.tar.gz -C /
```

### Configuring Containerd
Containerd is the container runtime used to run containers managed by Kubernetes in this course. In this lesson, we will configure a `systemd` service for containerd on both of our worker node servers. This containerd service will be used to run containerd as a component of each worker node. After completing this lesson, you should have a containerd configured to run as a `systemd` service on both workers.

- You can configure the containerd service like so. Run these commands on both worker nodes:
```
sudo mkdir -p /etc/containerd/
```

- Create the containerd config.toml:
```
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
    [plugins.cri.containerd.untrusted_workload_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
EOF
```

- Create the containerd unit file:
```
cat << EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

### Configuring Kubelet
Kubelet is the Kubernetes agent which runs on each worker node. Acting as a middleman between the Kubernetes control plane and the underlying container runtime, it coordinates the running of containers on the worker node. In this lesson, we will configure our `systemd` service for kubelet. After completing this lesson, you should have a `systemd` service configured and ready to run on each worker node.

- You can configure the kubelet service like so. Run these commands on both worker nodes.

- Set a `HOSTNAME` environment variable that will be used to generate your config files. Make sure you set the `HOSTNAME` appropriately for each worker node:
```
HOSTNAME=$(hostname)
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```

- Create the kubelet config file:
```
cat << EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS: 
  - "10.32.0.10"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

- Create the kubelet unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2 \\
  --hostname-override=${HOSTNAME} \\
  --allow-privileged=true
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configuring Kube-Proxy
Kube-proxy is an important component of each Kubernetes worker node. It is responsible for providing network routing to support Kubernetes networking components. In this lesson, we will configure our kube-proxy `systemd` service. Since this is the last of the three worker node services that we need to configure, we will also go ahead and start all of our worker node services once we're done. Finally, we will complete some steps to verify that our cluster is set up properly and functioning as expected so far. After completing this lesson, you should have two Kubernetes worker nodes up and running, and they should be able to successfully register themselves with the cluster.

- You can configure the kube-proxy service like so. Run these commands on both worker nodes:
```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

- Create the kube-proxy config file:
```
cat << EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```

- Create the kube-proxy unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

- Now you are ready to start up the worker node services! Run these:
```
sudo systemctl daemon-reload
sudo systemctl enable containerd kubelet kube-proxy
sudo systemctl start containerd kubelet kube-proxy
```

- Check the status of each service to make sure they are all `active (running)` on both worker nodes:
```
sudo systemctl status containerd kubelet kube-proxy
```

- Finally, verify that both workers have registered themselves with the cluster. Log in to one of your control nodes and run this:
```
kubectl get nodes
```

- You should see the hostnames for both worker nodes listed. Note that it is expected for them to be in the NotReady state at this point.
```
NAME                           STATUS     ROLES     AGE       VERSION
innaghiyev3c.mylabserver.com   NotReady   <none>    3m        v1.10.2
innaghiyev4c.mylabserver.com   NotReady   <none>    3m        v1.10.2
```

### Bootstrapping Kubernetes Worker Nodes
##### Additional Information and Resources
Your team wants to set up a new Kubernetes cluster. Control nodes have already been created and are ready to be used. However, no worker nodes have been set up. You have been given the task of setting up two worker nodes to be used as part of the new Kubernetes cluster.

##### Install the required packages.
- You can install the required packages like this. Make sure you install the packages on both worker nodes:
```
sudo apt-get -y install socat conntrack ipset
```

##### Download and install the necessary binaries.
- You can download and install the binaries like this:
```
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.0/crictl-v1.0.0-beta.0-linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-the-hard-way/runsc \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.1.0/containerd-1.1.0.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubelet

sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes

chmod +x kubectl kube-proxy kubelet runc.amd64 runsc

sudo mv runc.amd64 runc

sudo mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/

sudo tar -xvf crictl-v1.0.0-beta.0-linux-amd64.tar.gz -C /usr/local/bin/

sudo tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/

sudo tar -xvf containerd-1.1.0.linux-amd64.tar.gz -C /
```

##### Configure the containerd service.
- Configure the containerd service like this:
```
sudo mkdir -p /etc/containerd/
```

- Create the containerd `config.toml`.
```
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
    [plugins.cri.containerd.untrusted_workload_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
EOF
```

- Create the containerd unit file:
```
cat << EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

##### Configure the kubelet service.
- You can set up kubelet like this. Make sure you set `HOSTNAME` to `worker0` on the first worker node and `worker1` on the second. 
```
HOSTNAME=<worker1 or worker0, depending on the server>.mylabserver.com
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```

- Create the kubelet config file:
```
cat << EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

- Create the kubelet unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2 \\
  --hostname-override=${HOSTNAME} \\
  --allow-privileged=true
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

##### Configure the kube-proxy service.
- You can configure kube-proxy like this:
```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

- Create the kube-proxy config file:
```
cat << EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```

- Create the kube-proxy unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

##### Successfully start all of the services.
- Enable and start all of the services like this:
```
sudo systemctl daemon-reload
sudo systemctl enable containerd kubelet kube-proxy
sudo systemctl start containerd kubelet kube-proxy
```

- You can verify that the services are up and running like this:
```
sudo systemctl status containerd kubelet kube-proxy
```

- Make sure containerd, kubelet, and kube-proxy are all in the `active (running)` state on both worker nodes.

- Now make sure that both nodes are registering with the cluster. Log in to the control node and run this command:
```
kubectl get nodes --kubeconfig /home/cloud_user/admin.kubeconfig
```

- Make sure your two worker nodes appear. Note that they will likely not be in the `READY` state. For now, just make sure they both show up.
```
NAME                      STATUS     ROLES     AGE       VERSION
worker0.mylabserver.com   NotReady   <none>    55s       v1.10.2
worker1.mylabserver.com   NotReady   <none>    47s       v1.10.2
```

## Configuring kubectl for Remote Access
### Kubernetes Remote Access and kubectl
Kubectl is a powerful command-line tool that allows you to manage Kubernetes clusters. In order to manage the cluster from your machine, you will need to configure your local kubectl to connect to the remote cluster. This lesson discusses kubectl and introduces the rationale behind using it to manage the cluster remotely. It also discusses how a local kubectl installation fits into the larger overall cluster architecture. This will provide some background to help you understand the following lesson, which demonstrates how to implement this configuration.

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img18.png)

- You can find more information about kubectl in the official docs: https://kubernetes.io/docs/reference/kubectl/overview/

### Configuring Kubectl for Remote Access
There are a few steps to configuring a local kubectl installation for managing a remote cluster. This lesson will guide you through that process. After completing this lesson, you should have a local kubectl installation that is capable of running kubectl commands against your remote Kubernetes cluster.

- In a separate shell, open up an ssh tunnel to port 6443 on your Kubernetes API load balancer:
```
ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
```

- You can configure your local kubectl in your main shell like so. Set KUBERNETES_PUBLIC_ADDRESS to the public IP of your load balancer.
```
cd ~/kthw

kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://localhost:6443

kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem

kubectl config set-context kubernetes-the-hard-way \
  --cluster=kubernetes-the-hard-way \
  --user=admin

kubectl config use-context kubernetes-the-hard-way
```

- Verify that everything is working with:
```
kubectl get pods
kubectl get nodes
kubectl version
```

### Configuring Kubectl to Access a Remote Cluster
##### Additional Information and Resources

Your team has set up a Kubernetes cluster. You need to check the cluster to see what pods are currently running, but you are not in the office. You can use kubectl to interact with the cluster remotely, but first you need to configure your local kubectl with the proper kubeconfig. Your task is to configure kubectl so that you can interact with the cluster remotely, then use it to determine which pods are currently running on the cluster.

We’ll use the `Workspace` server to represent your local workstation. This is the server you will need to log into so that you can configure kubectl. Kubectl is already installed, so you do not need to install it manually.

##### Set the kubectl cluster data.
- You can set the cluster data like this. Be sure to set `KUBERNETES_PUBLIC_ADDRESS` to the public IP of your controller.
```
KUBERNETES_PUBLIC_ADDRESS=<Controller public IP>

kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443
```

##### Set the credentials for kubectl.
- The credentials are the certificate and key files `admin.pem` and `admin-key.pem`. These can be found in the home directory on the workspace server. You can set them up for kubectl like this:
```
kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem
```

##### Set the context for the cluster. 
- Set the context like this:
```
kubectl config set-context kubernetes-the-hard-way \
  --cluster=kubernetes-the-hard-way \
  --user=admin
```

##### Use the new kubectl context to check which pods are currently running on the cluster.
- Configure kubectl to use the new context like this:
```
kubectl config use-context kubernetes-the-hard-way
```

- Verify that everything is working with:
```
kubectl get pods
```

- Since there are no pods right now, you should get the following response:
```
No resources found.
```

- This indicates that kubectl is set up correctly to access the cluster!

## Networking
### The Kubernetes Networking Model
Kubernetes provides a powerful networking model which allows pods to communicate with one another over a virtual network, regardless of what host they are running on. However, Kubernetes networking can be confusing for those who are not familiar with the architecture behind networking in Kubernetes. This lesson introduces the Kubernetes networking model and provides some foundational knowledge about what it does. After completing this lesson, you will have a basic understanding of how networking works in Kubernetes, which will prepare you for the process of implementing networking in your cluster.
  
![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img19.png)

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img20.png)

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img21.png)

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img22.png)

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img23.png)
    
You can find more information on the Kubernetes networking model in the official docs: https://kubernetes.io/docs/concepts/cluster-administration/networking/

### Cluster Network Architecture
There are several components involved in implementing networking in a Kubernetes cluster. This lesson provides an overview of what we have already done in our cluster in order to implement networking, as well as the remaining steps that will be performed to implement a virtual cluster network. It also introduces Weave Net, the networking solution that we will use in this course. After completing this lesson, you will have an understanding of the networking configuration that we are implementing in our cluster, and you will be ready to install Weave Net.
  
![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img24.png)

You can find more information about Weave Net here: https://github.com/weaveworks/weave

### Installing Weave Net
We are now ready to set up networking in our Kubernetes cluster. This lesson guides you through the process of installing Weave Net in the cluster. It also shows you how to test your cluster network to make sure that everything is working as expected so far. After completing this lesson, you should have a functioning cluster network within your Kubernetes cluster.

- You can configure Weave Net like this:
- First, log in to both worker nodes and enable IP forwarding:
```
sudo sysctl net.ipv4.conf.all.forwarding=1
echo "net.ipv4.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.conf
```

- The remaining commands can be done using kubectl. To connect with kubectl, you can either log in to one of the control nodes and run kubectl there or open an SSH tunnel for port 6443 to the load balancer server and use kubectl locally.
- You can open the SSH tunnel by running this in a separate terminal. Leave the session open while you are working to keep the tunnel active:
```
ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
```

- Install Weave Net like this:
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.200.0.0/16"
```

- Now Weave Net is installed, but we need to test our network to make sure everything is working.
- First, make sure the Weave Net pods are up and running:
```
kubectl get pods -n kube-system
```

- This should return two Weave Net pods, and look something like this:
```
NAME              READY     STATUS    RESTARTS   AGE
weave-net-m69xq   2/2       Running   0          11s
weave-net-vmb2n   2/2       Running   0          11s
```

- Next, we want to test that pods can connect to each other and that they can connect to services. We will set up two Nginx pods and a service for those two pods. Then, we will create a busybox pod and use it to test connectivity to both Nginx pods and the service.
- First, create an Nginx deployment with 2 replicas:
```
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      run: nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF
```

- Next, create a service for that deployment so that we can test connectivity to services as well:
```
kubectl expose deployment/nginx
```

- Now let's start up another pod. We will use this pod to test our networking. We will test whether we can connect to the other pods and services from this pod.
```
kubectl run busybox --image=radial/busyboxplus:curl --command -- sleep 3600
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

- Now let's get the IP addresses of our two Nginx pods:
```
kubectl get ep nginx
```

- There should be two IP addresses listed under ENDPOINTS, for example:
```
NAME      ENDPOINTS                       AGE
nginx     10.200.0.2:80,10.200.128.1:80   50m
```

- Now let's make sure the busybox pod can connect to the Nginx pods on both of those IP addresses.
```
kubectl exec $POD_NAME -- curl <first nginx pod IP address>
kubectl exec $POD_NAME -- curl <second nginx pod IP address>
```

- Both commands should return some HTML with the title "Welcome to Nginx!" This means that we can successfully connect to other pods.
- Now let's verify that we can connect to services.
```
kubectl get svc
```

- This should display the IP address for our Nginx service. For example, in this case, the IP is 10.32.0.54:
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP   1h
nginx        ClusterIP   10.32.0.54   <none>        80/TCP    53m
```

- Let's see if we can access the service from the busybox pod!
```
kubectl exec $POD_NAME -- curl <nginx service IP address>
```

- This should also return HTML with the title "Welcome to Nginx!"

- This means that we have successfully reached the Nginx service from inside a pod and that our networking configuration is working!

### Cleanup
Now that we have networking set up in the cluster, we need to clean up the objects that were created in order to test the networking. These object could get in the way or become confusing in later lessons, so it is a good idea to remove them from the cluster before proceeding. After completing this lesson, your networking should still be in place, but the pods and services that were used to test it will be cleaned up.
  
To do this, you will need to use kubectl. To connect with kubectl, you can either log in to one of the control nodes and run kubectl there or open an SSH tunnel for port 6443 to the load balancer server and use kubectl locally.
  
- You can open the SSH tunnel by running this in a separate terminal. Leave the session open while you are working to keep the tunnel active:
```
ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
```

- You can clean up the testing objects from the previous lesson like so:
```
kubectl delete deployment busybox
kubectl delete deployment nginx
kubectl delete svc nginx
```

### Setting Up Kubernetes Networking with Weave Net
##### Additional Information and Resources
Your team is configuring a new Kubernetes cluster to run your company’s new online store. The controller and worker nodes have been set up, but some of the pods in your infrastructure will need to communicate with each other. Therefore, you need to configure Kubernetes networking. In this learning activity, you will implement networking in a Kubernetes cluster using Weave Net.

##### Enable IP forwarding on all worker nodes.
- In order for Weave Net to work, you need to make sure IP forwarding is enabled on the worker nodes. Enable it by running the following on both workers:
```
sudo sysctl net.ipv4.conf.all.forwarding=1
echo "net.ipv4.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.conf
```

##### Install Weave Net in the cluster.
- Do the following on the controller server:
1. Install Weave Net using a configuration from Weaveworks like this:
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.200.0.0/16"
```

2. Verify that everything is working:
```
kubectl get pods -n kube-system
```
- This should return two `weave-net` pods and look something like this:
```
NAME              READY     STATUS    RESTARTS   AGE
weave-net-m69xq   2/2       Running   0          11s
weave-net-vmb2n   2/2       Running   0          11s
```

3. Spin up some pods to test the networking functionality:
  
a. First, create an Nginx deployment with 2 replicas:

```
  cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx
  spec:
    selector:
      matchLabels:
        run: nginx
    replicas: 2
    template:
      metadata:
        labels:
          run: nginx
      spec:
        containers:
        - name: my-nginx
          image: nginx
          ports:
          - containerPort: 80
  EOF
```

b. Next, create a service for that deployment so that we can test connectivity to services as well:
```
kubectl expose deployment/nginx
```
  
c. Start up another pod. We will use this pod to test our networking. We will test whether we can connect to the other pods and services from this pod.
```
kubectl run busybox --image=radial/busyboxplus:curl --command -- sleep 3600
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

d. Get the IP addresses of our two `nginx` pods:
```
kubectl get ep nginx
```
  
There should be two IP addresses listed under `ENDPOINTS`. For example:
```
NAME      ENDPOINTS                       AGE
nginx     10.200.0.2:80,10.200.128.1:80   50m
```

4. Make sure the `busybox` pod can connect to the `nginx` pods on both of those IP addresses.
```
kubectl exec $POD_NAME -- curl <first nginx pod IP address>
kubectl exec $POD_NAME -- curl <second nginx pod IP address>
```
  
Both commands should return some HTML with the title `"Welcome to Nginx!"` This means that we can successfully connect to other pods.

5. Now let's verify that we can connect to services.
```
kubectl get svc
```
  
This should display the IP address for our Nginx service. For example, in this case, the IP is `10.32.0.54`:
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP   1h
nginx        ClusterIP   10.32.0.54   <none>        80/TCP    53m
```

6. Check that we can access the service from the busybox pod.
```
kubectl exec $POD_NAME -- curl <nginx service IP address>
```
  
This should also return HTML with the title `"Welcome to nginx!"`
  
Getting this response means that we have successfully reached the Nginx service from inside a pod and that our networking configuration is working!

## Deploying the DNS Cluster Add-on
### DNS In a Kubernetes Pod Network
In this section, we will be implementing DNS within the Kubernetes cluster. This lesson provides some background which will help you understand what DNS does within a Kubernetes cluster and what it is used for. After completing this lesson, you will have a basic understanding of how DNS works in a Kubernetes cluster, and you will understand what we are trying to accomplish as we configure DNS in the cluster.

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img25.png)

You can find more information about Kubernetes DNS here: https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

### Deploying Kube-dns to the Cluster
Kube-dns is an easy-to-use solution for providing DNS service in a Kubernetes cluster. This lesson guides you through the process of installing kube-dns in your cluster, as well as testing your DNS setup to make sure that it is working. After completing this lesson, you should have a working kube-dns installation in your cluster, and pods should be able to successfully use your DNS.
  
To install and test kube-dns, you will need to use `kubectl`. To connect with `kubectl`, you can either log in to one of the control nodes and run `kubectl` there, or open an SSH tunnel for port 6443 to the load balancer server and use `kubectl` locally.
  
- You can open the SSH tunnel by running this in a separate terminal. Leave the session open while you are working to keep the tunnel active:
```
ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
```

- You can install kube-dns like so:
```
kubectl create -f https://storage.googleapis.com/kubernetes-the-hard-way/kube-dns.yaml
```

- Verify that the kube-dns pod starts up correctly:
```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

- You should get output showing the kube-dns pod. It should look something like this:
```
NAME                        READY     STATUS    RESTARTS   AGE
kube-dns-598d7bf7d4-spbmj   3/3       Running   0          36s
```
  
Make sure that 3/3 containers are ready, and that the pod has a status of `Running`. It may take a moment for the pod to be fully up and running, so if READY is not 3/3 at first, check again after a few moments.

- Now let's test our kube-dns installation by doing a DNS lookup from within a pod. First, we need to start up a pod that we can use for testing:
```
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

- Next, run an `nslookup` from inside the busybox container:
```
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

- You should get output that looks something like this:
```
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```

  If `nslookup` succeeds, then your kube-dns installation is working!

- Once you are done, it's probably a good idea to clean up the the objects that were created for testing:
```
kubectl delete deployment busybox
```

### Deploying kube-dns in a Kubernetes Cluster
##### Additional Information and Resources
Your team has set up a new Kubernetes cluster. However, they want to provide a simple web service as a backend microservice in the cluster. In order to do so, they need a DNS set up within the Kubernetes cluster. Your task is to provide DNS service within the Kubernetes cluster using `kube-dns`.
  
You can access a pre-configured `kubectl` by logging in to the controller server.

##### Deploy kube-dns to the cluster.
- You can deploy kube-dns to the cluster like so:
```
kubectl create -f https://storage.googleapis.com/kubernetes-the-hard-way/kube-dns.yaml
```

- Verify that the kube-dns pod starts up correctly:
```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

- You should get output showing the kube-dns pod. It should look something like this:
```
NAME                        READY     STATUS    RESTARTS   AGE
kube-dns-598d7bf7d4-spbmj   3/3       Running   0          36s
```
  
Make sure that 3/3 containers are ready and that the pod has a status of `Running`. It may take a moment for the pod to be fully up and running, so if READY is not 3/3 at first, check again after a few moments.

##### Test the DNS by creating a service, and perform a DNS lookup for service from another pod using the service name.

- Test the DNS by creating a service, and access the service from another pod using the service name.

- Create a simple service like so:
```
kubectl run nginx --image=nginx
kubectl expose deployment nginx --port 80
```

- Get a list of services:
```
kubectl get svc
```
  
You should see the `nginx` service listed.

- Spin up a busybox pod for testing and get the pod name:
```
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

- Perform a DNS lookup of the `nginx` service from within the busybox pod to verify that DNS is working:
```
kubectl exec $POD_NAME -- nslookup nginx
```

- You should get output that looks like this:
```
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx
Address 1: 10.32.0.248 nginx.default.svc.cluster.local
```


## Smoke Test
### Smoke Testing the Cluster
Congratulations! You have successfully set up a new Kubernetes cluster from scratch! However, at this stage it is a good idea to verify that the cluster is working fully as expected. In this section, we will be running some basic smoke tests to verify that the cluster is set up correctly. This lesson discusses the smoke tests that will be done in this section, and after completing this lesson you will be ready to begin the process of performing these tests against your cluster.

![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img26.png)

### Smoke Testing Data Encryption
Earlier in this course, we set up a data encryption config to allow Kubernetes to encrypt sensitive data. In this lesson, we will smoke test that functionality by creating some secret data and verifying that it is stored in an encrypted format in etcd. After completing this lesson, you will have verified that your cluster can successfully encrypt sensitive data.
  
![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img27.png)
  
For this lesson, you will need to connect to cluster using `kubectl`. You can log in to one of your controller servers and use `kubectl` there, or you can use `kubectl` from your local machine. To use `kubectl` from your local machine, you will need to open an SSH tunnel. You can open the SSH tunnel by running this in a separate terminal. Leave the session open while you are working to keep the tunnel active:
```
ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
```

- Create a test secret:
```
kubectl create secret generic kubernetes-the-hard-way --from-literal="mykey=mydata"
```

- Log in to one of your controller servers, and get the raw data for the test secret from etcd:
```
sudo ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C
```

- Your output should look something like this:
```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a fc 21 ee dc e5 84 8a  |:v1:key1:.!.....|
00000050  53 8e fd a9 72 a8 75 25  65 30 55 0e 72 43 1f 20  |S...r.u%e0U.rC. |
00000060  9f 07 15 4f 69 8a 79 a4  70 62 e9 ab f9 14 93 2e  |...Oi.y.pb......|
00000070  e5 59 3f ab a7 b2 d8 d6  05 84 84 aa c3 6f 8d 5c  |.Y?..........o.\|
00000080  09 7a 2f 82 81 b5 d5 ec  ba c7 23 34 46 d9 43 02  |.z/.......#4F.C.|
00000090  88 93 57 26 66 da 4e 8e  5c 24 44 6e 3e ec 9c 8e  |..W&f.N.\$Dn>...|
000000a0  83 ff 40 9a fb 94 07 3c  08 52 0e 77 50 81 c9 d0  |..@....<.R.wP...|
000000b0  b7 30 68 ba b1 b3 26 eb  b1 9f 3f f1 d7 76 86 09  |.0h...&...?..v..|
000000c0  d8 14 02 12 09 30 b0 60  b2 ad dc bb cf f5 77 e0  |.....0.`......w.|
000000d0  4f 0b 1f 74 79 c1 e7 20  1d 32 b2 68 01 19 93 fc  |O..ty.. .2.h....|
000000e0  f5 c8 8b 0b 16 7b 4f c2  6a 0a                    |.....{O.j.|
000000ea
```
  
Look for `k8s:enc:aescbc:v1:key1` on the right of the output to verify that the data is stored in an encrypted format!

### Smoke Testing Deployments
Deployments are one of the powerful orchestration tools offered by Kubernetes. In this lesson, we will make sure that deployments are working in our cluster. We will verify that we can create a deployment, and that the deployment is able to successfully stand up a new pod and container.
  
![img](https://github.com/Bes0n/KubernetestheHardWay/blob/master/images/img28.png)

For this lesson, you will need to connect to cluster using `kubectl`. You can log in to one of your controller server and use `kubectl` there, or you can use `kubectl` from your local machine. To use `kubectl` from your local machine, you will need to open an SSH tunnel. You can open the SSH tunnel by running this in a separate terminal. Leave the session open while you are working to keep the tunnel active:
```
ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
```

- Create a a simple nginx deployment:
```
kubectl run nginx --image=nginx
```

- Verify that the deployment created a pod and that the pod is running:
```
kubectl get pods -l run=nginx
```

- Verify that the output looks something like this:
```
NAME                     READY     STATUS    RESTARTS   AGE
nginx-65899c769f-9xnqm   1/1       Running   0          30s
```
  
The pod should have a STATUS of `Running` with 1/1 containers READY.