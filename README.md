# Forschungsprojekt 2025

There is a Makefile with various commands. Use `make help` to get started.

## Containerlab Kubernetes Quickstart
https://containerlab.dev/manual/clabernetes/quickstart/

## Multipass
https://multipass.run/docs

### Create a containerlab instance

```bash
multipass launch --name containerlab --cpus 2 --mem 4G --disk 20G
```

### Set containerlab ram
```bash
multipass set local.containerlab.memory
```

### Set containerlab disk
```bash
multipass set local.containerlab.disk
```

### Set containerlab cpus
```bash
multipass set local.containerlab.cpus
```

### Start containerlab instance
```bash
multipass start containerlab
```
### Stop containerlab instance
```bash
multipass stop containerlab
```

### Shell into containerlab instance
```bash
multipass shell containerlab
```

## kind
https://kind.sigs.k8s.io/docs/user/quick-start/

```bash
kind create cluster --config kind-config.yaml
```

## kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```bash
kubectl get nodes
kubectl get pods
kubectl get pods --all-namespaces
kubectl get Topology
kubectl get namespaces
kubectl get ns
kubectl cluster-info
kubectl cluster-info dump
kubectl get all
kubectl get all --all-namespaces
kubectl get all -o wide
```

The above commands can take the `-n <namespace>` flag to specify a namespace.
## Helm
https://helm.sh/docs/intro/install/

```bash
helm install {clabernetes or other name} oci://ghcr.io/srl-labs/clabernetes/clabernetes
```

## Clabverter

### Setup

```bash
alias clabverter='sudo docker run --user $(id -u) \
    -v $(pwd):/clabernetes/work --rm \
    ghcr.io/srl-labs/clabernetes/clabverter'
```

### ARM Architecture caveat
In multipass ubuntu on mac m1, the architecture is `aarch64` (ARM64).

To use clabverter you need to use qemu to emulate the x86_64 architecture.
```bash
sudo apt install qemu binfmt-support qemu-user-static
```

Set in docker to use qemu:
```bash
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

## References
https://github.com/srl-labs/srlinux-vlan-handling-lab/

## Misc.
### k3d
https://k3d.io/v5.4.0/usage/quick-start/
