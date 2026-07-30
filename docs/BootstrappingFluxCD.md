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