## Ingress Setup

For this step, I chose Traefik.

> [!NOTE]
> To install traefik, I created a separate repository that can be found [here](https://github.com/stefanescualexandrumihai/TechnicalChallenge-Traefik). I used umbrella chart pattern to install the traefik official helm chart. This was done in case we need additional components that are not included in the chart and to cover this pattern too.

In the [infrastructure repository](https://github.com/stefanescualexandrumihai/TechnicalChallenge) we have an "entrypoint" for `admin apps` in each cluster (`infrastructure.yaml`) alongside the `tenant applications entrypoints`.

e.g. `path: ./infrastructure/clusters/tc-dev-euc1`

```text
clusters/
├── tc-dev-euc1/
│   ├── cluster-vars.yaml       # cluster specific variables such as env, region, etc.
│   ├── infrastructure.yaml     # entrypoint to the admin apps
│   └── ...
├── tc-dev-jpn3/
├── tc-prod-irl1/
└── ...
```

Since each `infrastructure.yaml` from `clusters/CLUSTER_NAME` is a `Kustomization` from `kustomize.toolkit.fluxcd.io/v1`, we will define in each `./infrastructure/clusters/CLUSTER_NAME` folder a `kustomization.yaml` file.
That file will point to another place (`../../environment/{ENV}`) and will carry the `cluster` specific override values for each controller (in case there are) using `patches`.

> [!NOTE]
> This is how kustomize tool overrides values, identifying the object by apiVersion, kind, metadata.name, metadata.namespace and applying a strategic merge.

In `../../environment/{ENV}` we will find another `kustomization.yaml` file that points to another place (`../../controllers`) and carries the `environment` specific override values for each controller (in case there are) using `patches`.

In `../../controllers` we could find our controllers such as keycloak, traefik, prometheus, etc. and this could be marked as our "leaf". Another `kustomization.yaml` is present in every controller folder and holds the manifests that will be built by kustomize-controller using `resources`. This is where the nested structure ends.

For traefik:

```text
controllers/
├── traefik/
│   ├── kustomization.yaml       # Holds the resources that should be built by kustomize
│   ├── source.yaml              # the git repository where k8s manifests reside
│   ├── namespace.yaml           # k8s namespace where traefik will reside
│   └── helmrelease.yaml         # the HelmRelease manifest for traefik
├── keycloak/
├── metrics-server/
└── ...
```

> [!IMPORTANT]
> The above nested structure is used to leverage the overrides capability of kustomize. Therefore, the final HelmRelease object will contain the strategic merge output from this flow. If I define some override values in `environment/dev` for traefik, these values will be applied to the helm chart for all dev clusters. However, if I do the same in `infrastructure/clusters/CLUSTER_NAME`, these values will override the environment.
