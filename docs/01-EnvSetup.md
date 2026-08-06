# Local kind cluster

In order to accomplish this task, we need to install a local k8s cluster.
The easiest way is to use a tool that emulates a k8s cluster using docker containers.

## Prerequisites

docker, kind, kubectl, helm

```bash
brew install --cask docker
brew install kind
brew install kubectl
brew install helm
```

## Install

```bash
kind create cluster --config kind/kind-config.yaml
kubectl get nodes
```

Output

```bash
kubectl get nodes
NAME                               STATUS   ROLES           AGE     VERSION
technicalchallenge-control-plane   Ready    control-plane   4m15s   v1.36.1
technicalchallenge-worker          Ready    <none>          4m1s    v1.36.1
technicalchallenge-worker2         Ready    <none>          4m1s    v1.36.1
```

> [!NOTE]
> To access the ingress running in the cluster from localhost, we need to map
> the interface 127.0.0.1:PORT to a specific port on the container. Therefore, all traffic
> sent to localhost:PORT on the Mac is forwarded to the container port.
