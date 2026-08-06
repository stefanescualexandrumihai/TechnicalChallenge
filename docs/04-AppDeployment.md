## Application Deployment

> [!NOTE]
> For this application, I created a separate git repository that can be found [here](https://github.com/stefanescualexandrumihai/TechnicalChallenge-loadtester).
> This repo contains vanilla k8s manifests. However, in order to create a solution for multiple deployments to different environments, the repo is structured based on kustomize pattern.

```text
/
├── base/
│   ├── kustomization.yaml       # kustomization file that lists all resources to be built
│   ├── deployment.yaml          # app deployment
│   ├── service.yaml             # app service
│   ├── ingress.yaml             # app ingress
│   └── namespace.yaml           # app namespace
└── overlays/
    ├── dev/
    │   ├── deployment-patch.yaml     # deployment patch
    │   └── kustomization.yaml        # points to base and adds the patches from dev deployment-patch
    └── prod/
        ├── deployment-patch.yaml     # deployment patch
        └── kustomization.yaml        # points to base and adds the patches from prod deployment-patch
```

After the repo is defined, we need to add an "entry" to [Infrastructure Repo](https://github.com/stefanescualexandrumihai/TechnicalChallenge). In `./apps`, there is a new folder created called `tc-loadtester`.

In that folder, we'll define the following:

```text
apps/
├── tc-loadtester/
│   ├── kustomization.yaml             # specific to kustomize tool. Lists the resources that should be built
│   ├── source.yaml                    # the git repository where k8s manifests reside
│   └── flux-kustomization.yaml        # flux Kustomization for the kustomize repo above
└── ...
```

Since the flux instance reconciles `clusters/tc-dev-euc1`, we'll create an "entrypoint" for our app in that cluster. Under `clusters/tc-dev-euc1`, there will be a file called `tc-loadtester.yaml` which contains a `Kustomization` manifest that is reconciled by kustomize-controller. That manifest points to `./apps/tc-loadtester`, defined previously.

> [!IMPORTANT]
> There is an extra feature added here.
> In `apps/tc-loadtester/flux-kustomization.yaml`, we have `path: ./overlays/${ENV}`.
> In `clusters/tc-dev-euc1/tc-loadtester.yaml`, we have:
>
> ```yaml
> spec:
>   postBuild:
>     substituteFrom:
>       - kind: ConfigMap
>         name: cluster-vars
> ```
>
> This allows us to use the cluster `ENV`, defined once in the `/clusters/tc-dev-euc1/cluster-vars.yaml`. If we choose to deploy the same app on a production cluster, we just need to make sure that in the app git repo, the `overlays/prod` folder has the same name as the cluster env (prod -> prod).

## Test

In order to test you can execute (from your PC):

```bash
curl http://loadtester.localhost:8080/

{"status":"ok"}
```
