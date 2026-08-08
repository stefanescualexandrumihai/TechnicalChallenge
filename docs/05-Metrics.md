## Setting Up Metrics

For metrics-server I chose to use the official chart.
Therefore I added a new entry in `/infrastructure/controllers/metrics-server`.

```text
controllers/
├── metrics-server/
│   ├── kustomization.yaml       # Holds the resources that should be built by kustomize
│   ├── source.yaml              # the helm repository that hosts the chart
│   └── helmrelease.yaml         # helm release
└── ...
```

The `kustomization.yaml` lists the resources within that folder.
The `source.yaml` contains a `HelmRepository` manifest that defines the source of the chart.
After we have the source, I defined the helmrelease manifest that contains the chart name and attributes used by helm-controller.
Within `/controllers` I have a `kustomization.yaml` file too, which is referenced by all environments. Therefore, I need to add the metrics-server resource here, too.

```yaml
resources:
  - traefik
  - metrics-server
```

Since FluxInstance reconciles `/clusters/tc-dev-euc1`, we need to check the path to make sure that the new controller is included. Therefore, the first thing to check is the `kustomization.yaml` (within that folder). That `kustomization.yaml` points to `../../environment/dev`. However, I want to add a specific override for this particular cluster, therefore I'll add a new file in this folder, called `metrics-server.yaml`, with the specific override.

```yaml
    - --kubelet-insecure-tls
```

> [!WARNING]
> The kubelet is the server and metrics-server is the client. Since `serverTLSBootstrap` is
> false, the kubelet serving certificate is self-signed. The encryption is there, but the
> validation doesn't exist.

In `/infrastructure/environment/dev` I do not have to add override values, and the `kustomization.yaml` from that folder points directly to `../../controllers`.

The version of the chart is written in `/clusters/tc-dev-euc1/cluster-vars.yaml` and used in `/infrastructure/controllers/metrics-server/helmrelease.yaml`.

## Test

How to check if metrics-server is working as expected:

```bash
kubectl top pods -n kube-system
NAME                                                       CPU(cores)   MEMORY(bytes)
coredns-589f44dc88-66brt                                   1m           30Mi
coredns-589f44dc88-lhm2h                                   1m           25Mi
etcd-technicalchallenge-control-plane                      14m          86Mi
kindnet-4gn27                                              1m           21Mi
kindnet-8t2lz                                              1m           23Mi
kindnet-mvljd                                              1m           26Mi
kube-apiserver-technicalchallenge-control-plane            29m          516Mi
kube-controller-manager-technicalchallenge-control-plane   7m           99Mi
kube-proxy-l8c5z                                           1m           32Mi
kube-proxy-vpnsl                                           1m           35Mi
kube-proxy-wsmpm                                           1m           33Mi
kube-scheduler-technicalchallenge-control-plane            4m           44Mi
metrics-server-658d9f849f-xwm9m                            2m           36Mi
```
