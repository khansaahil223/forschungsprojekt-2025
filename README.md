# Forschungsprojekt 2025

## Abstract

Network Digital Twins (DTs) help monitor, maintain and test network infrastructures. 
They have been implemented using various approaches. 
In this work, how DTs can be implemented using a combination of containerlabs and kubernetes is evaluated. 
Two approaches are presented. 
One is a local approach and one is a cloud (openstack) based approach. Various network topologies and their latencies on both setups are tabulated. 
Both approaches are evaluated in terms of ease of setup, network infrastructure requirements and network latencies.

# Local setup
## Prerequisites
For the local setup following prerequisites must be available on the system. 
- Docker 
- kind (Kubernetes in Docker)
- helm
- kubectl (Kubernetes CLI)

## Automation Script
After the prerequisites are installed, there is a simple bash script available in the [local](local/deploy-and-measure) folder. 
It must be run from within the local folder and has a simple usage described below. 
It will setup the selected Kubernetes cluster, clabvert the topology and measure the latencies between client1 and other clients.

```bash 
./deploy-and-measure {Cluster} {Topology}
```
Following clusters are available
- 1worker
- 2workers
- 3workers
- 4workers
- 2workers-2controlplanes
- 4workers-2controlplanes

Following topologies are available
- 1router-2clients
- 1router-4clients
- 2router-2clients
- 2router-4clients

# Cloud Setup
Thanks to the Kubernetes deployment setup from Professor Rieger at https://github.com/srieger1/terraform-openstack-rke2, it is very easy to setup Kubernetes on OpenStack. 
The terraform-openstack-rke2 is added as a submodule in this repository. 
See below for setup instructions.
Refer the README of terraform-openstack-rke2 for more details on the Openstack setup. 

The submodule is located in `cloud/terraform-openstack-rke2` folder.
This folder will be empty if this repository is cloned with a simple `git clone` command.
To clone the submodule, run the following commands.
```bash
git submodule init
git submodule update
```

After running the commands, the `cloud/terraform-openstack-rke2` will contain all the files including README.md. 
We use the hs-fulda example located in the [cloud/terraform-openstack-rke2/examples/hs-fulda](cloud/terraform-openstack-rke2/examples/hs-fulda/) folder.

## Clabernetes
After setup via terraform, ensure to use the correct KUBECONFIG via
```bash 
export KUBECONFIG=/path/to/*-k8s.rke2.yaml
```
[Clabverted](https://containerlab.dev/manual/clabernetes/quickstart/#clabverter) Kubernetes manifests can be found in [cloud/topologies](cloud/topologies).
In order to deploy these use the following command.
```bash 
kubectl apply -f cloud/topologies/{topology}.yaml
```
After this a list of pods and there IP addresses can be listed via
```bash 
kubectl get pods -n c9s-vlan -o wide
````
The pods can be accessed via SSH, refer the Openstack console to setup SSH via private key. 

The latencies are obtained via the ping command. 
```bash 
ping -q -i 0.1 -c 100 {IP_ADDRESS}
```