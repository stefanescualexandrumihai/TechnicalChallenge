# Local kind cluster
Local Kubernetes cluster.

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
kind create cluster --config kind-config.yaml
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
