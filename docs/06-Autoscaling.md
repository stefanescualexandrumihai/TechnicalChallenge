## Autoscaling

To handle the case where a large number of requests hit the application, we need to spin up new pods in order to distribute the load. The `HorizontalPodAutoscaler` (HPA) is the solution here.

This relies on the metrics-server deployed in [05-Metrics](./05-Metrics.md), which the HPA queries for pod CPU usage.

Since the `HorizontalPodAutoscaler` object is built in, we only create a single manifest that defines the scaling behaviour. The target is the `tc-loadtester` deployment.

The manifest can be found [here](https://github.com/stefanescualexandrumihai/TechnicalChallenge-loadtester/blob/main/base/hpa.yaml).

```yaml
minReplicas: 1
maxReplicas: 10
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

The HPA sets the number of replicas based on the rules above, keeping the average CPU usage across the pods at around 50%.

> [!IMPORTANT]
> With `type: Utilization`, `averageUtilization` is a percentage of the container's `resources.requests.cpu`, not its limit. The [deployment](https://github.com/stefanescualexandrumihai/TechnicalChallenge-loadtester/blob/main/base/deployment.yaml) therefore sets `resources.requests.cpu` (here `100m`). Without a request, the HPA cannot compute a percentage, reports `<unknown>`, and never scales.

In order to let the HPA do its work instead of a static replica count, the [tc-loadtester deployment](https://github.com/stefanescualexandrumihai/TechnicalChallenge-loadtester/blob/main/base/deployment.yaml) MUST NOT set `spec.replicas`. Because the manifests are deployed through Flux, there is an extra reason to leave it unset: otherwise Flux would reconcile it back to the static value on every sync and fight the HPA.

## Test

```bash
kubectl get hpa -n tc-loadtester
NAME            REFERENCE                  TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
tc-loadtester   Deployment/tc-loadtester   12%/50%   1         10        1          2m
```

If the `TARGETS` column shows `<unknown>/50%`, either the CPU request is missing from the deployment or the metrics-server is not ready.

If you send requests to the `/burn` endpoint and wait a few seconds, you can watch the HPA scale up the number of pods.

```bash
hey -c 10 -z 1m .../burn
```

```bash
kubectl top pods -n tc-loadtester
NAME                             CPU(cores)   MEMORY(bytes)
tc-loadtester-75d6d44ccb-7j86p   1m           46Mi    
tc-loadtester-75d6d44ccb-g4j8q   965m         46Mi
tc-loadtester-75d6d44ccb-hltck   38m          38Mi
tc-loadtester-75d6d44ccb-mjtpj   2m           38Mi
...        
```

```bash
kubectl get events -n tc-loadtester
...
46s         Normal    Created             pod/tc-loadtester-75d6d44ccb-zjd42      Container created
46s         Normal    Started             pod/tc-loadtester-75d6d44ccb-zjd42      Container started
62s         Normal    SuccessfulCreate    replicaset/tc-loadtester-75d6d44ccb     Created pod: tc-loadtester-75d6d44ccb-7j86p
62s         Normal    SuccessfulCreate    replicaset/tc-loadtester-75d6d44ccb     Created pod: tc-loadtester-75d6d44ccb-sgng8
62s         Normal    SuccessfulCreate    replicaset/tc-loadtester-75d6d44ccb     Created pod: tc-loadtester-75d6d44ccb-tdg82
47s         Normal    SuccessfulCreate    replicaset/tc-loadtester-75d6d44ccb     Created pod: tc-loadtester-75d6d44ccb-r6x22
47s         Normal    SuccessfulCreate    replicaset/tc-loadtester-75d6d44ccb     Created pod: tc-loadtester-75d6d44ccb-zjd42
47s         Normal    SuccessfulCreate    replicaset/tc-loadtester-75d6d44ccb     Created pod: tc-loadtester-75d6d44ccb-hltck
...
```