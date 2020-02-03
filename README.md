# KubernetestheHardWay
Kubernetes the Hard Way

- [Getting Started](#getting-started)
    - [What Will the Kubernetes Cluster Architecture Look Like?](#what-will-the-kubernetes-cluster-architecture-look-like)
    - [Setting Up Your Cloud Servers](#setting-up-your-cloud-servers)
    - [Client Tools](#client-tools)
- [Provisioning the CA and Generating TLS Certificates](#provisioning-the-ca-and-generating-tls-certificates)
    - [Why Do We Need a CA and TLS Certificates?](#why-do-we-need a-ca-and-tls-certificates)
    - [Provisioning the Certificate Authority](#provisioning-the-certificate-authority)
    - [](#)
    - [](#)
    - [](#)
    - [](#)
    - [](#)

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