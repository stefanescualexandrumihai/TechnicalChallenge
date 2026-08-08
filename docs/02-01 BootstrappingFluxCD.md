## Install

To install Flux Operator, use [Install the Flux Operator](https://fluxcd.io/flux/installation/#install-the-flux-operator).

```bash
helm install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator \
  --namespace flux-system \
  --create-namespace
```

Output

```bash
Pulled: ghcr.io/controlplaneio-fluxcd/charts/flux-operator:0.57.0
Digest: sha256:35105642acae3aceaf6a22e986653f766827c46e72179a9fec23214ec5f5317c
NAME: flux-operator
LAST DEPLOYED: Thu Jul 30 12:31:15 2026
NAMESPACE: flux-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Documentation at https://fluxoperator.dev/docs/
```

## Setup the FluxInstance

At this point, we have the flux-operator installed.

```bash
kubectl get pods -n flux-system
NAME                                           READY   STATUS    RESTARTS      AGE
flux-operator-5f68db7f94-4ncqp                 1/1     Running   6 (25h ago)   6d22h
```

> [!NOTE]
> This operator reconciles the **FluxInstance** resource, which is the core of the flux workflow.
> This FluxInstance installs the necessary controllers such as: `helm-controller`, `kustomize-controller`, `source-controller`, etc.
>
> The manifest can be found here: [TechnicalChallenge/flux/tc-dev-euc1-flux-instance.yaml](https://github.com/stefanescualexandrumihai/TechnicalChallenge/blob/main/flux/tc-dev-euc1-flux-instance.yaml)
>
> FluxInstance configures synchronization with the Git repo specified in `.spec.sync.url`.

```bash
flux get sources git
NAME                    REVISION                        SUSPENDED       READY   MESSAGE
flux-system             refs/heads/main@sha1:4d396509   False           True    stored artifact for revision 'refs/heads/main@sha1:4d396509'
```

```bash
flux get kustomizations
NAME                    REVISION                        SUSPENDED       READY   MESSAGE
flux-system             refs/heads/main@sha1:4d396509   False           True    Applied revision: refs/heads/main@sha1:4d396509
```
