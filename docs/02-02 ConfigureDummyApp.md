## Configure the Infrastructure Repo

> [!NOTE]
> To avoid duplication and maintaining multiple Git repositories for different configurations, we organize our manifests using a monorepo structure. This approach provides a single source of truth while supporting environment-specific overrides, defining which manifests are deployed to each cluster, and versioning changes independently for each environment or cluster.
>
> The proposed repository structure is shown below:

```text
clusters/
├── tc-dev-euc1/
│   ├── cluster-vars.yaml       # cluster specific variables such as env, region, etc.
│   ├── infrastructure.yaml     # entrypoint to the admin apps
│   ├── tenant-app1.yaml        # entrypoint to the tenant app
│   └── tenant-app2.yaml        # entrypoint to the tenant app
├── tc-dev-jpn3/
├── tc-prod-irl1/
└── ...
```

> [!NOTE]
> Each `tenant-app*.yaml` contains a `Kustomization` object (from `kustomize.toolkit.fluxcd.io/v1`, specific to Flux). That manifest is reconciled by kustomize-controller and points to `./apps/tenant-app`.
>
> The reconciliation loop executes a `kustomize build` command and the output is applied within the cluster. Therefore, each tenant folder (in `apps`) will have a `kustomization.yaml` (`kustomize.config.k8s.io/v1beta1` - specific to kustomize tool, not flux) file that lists all the yaml manifests that should be built by kustomize-controller.

```bash
flux get kustomizations tc-app
NAME    REVISION                        SUSPENDED       READY   MESSAGE
tc-app  refs/heads/main@sha1:4d396509   False           True    Applied revision: refs/heads/main@sha1:4d396509
```

```text
apps/
├── tenant-app1/
│   ├── kustomization.yaml             # specific to kustomize tool
│   ├── source.yaml                    # the git repository where k8s manifests reside
│   ├── helmrelease.yaml               # if the git repository from above is a helm chart
│   ├── flux-kustomization.yaml        # OR, if the git repository from above is a kustomize repo
│   └── others.yaml                    # other manifests if necessary
├── tenant-app2/
├── tenant-app3/
└── ...
```

```bash
flux get helmreleases -n tc-app
NAME    REVISION                SUSPENDED       READY   MESSAGE
tc-app  1.0.0+db699f04b8fd      False           True    Helm upgrade succeeded for release tc-app/tc-app.v3 with chart tc-app@1.0.0+db699f04b8fd
```
