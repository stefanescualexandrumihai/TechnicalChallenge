## Protect your app via Keycloak

For keycloak I chose to use the official chart.
Therefore I added a new entry in `/infrastructure/controllers/keycloak`.

```text
controllers/
├── keycloak/
│   ├── kustomization.yaml       # Holds the resources that should be built by kustomize
│   ├── source.yaml              # the helm repository that hosts the chart
│   ├── namespace.yaml           # k8s namespace
│   └── helmrelease.yaml         # helm release
└── ...
```

The `kustomization.yaml` lists the resources within that folder.
The `source.yaml` contains a `HelmRepository` manifest that defines the source of the chart.
After we have the source, I defined the helmrelease manifest that contains the chart name and attributes used by helm-controller.
Within `/controllers` I have a `kustomization.yaml` file too, which is referenced by all environments. Therefore, I need to add the keycloak resource here, too.

```yaml
resources:
  - traefik
  - metrics-server
  - keycloak
```

Since FluxInstance reconciles `/clusters/tc-dev-euc1`, we need to check the path to make sure that the new controller is included. Therefore, the first thing to check is the `kustomization.yaml` (within that folder). That `kustomization.yaml` points to `../../environment/dev`. However, I want to add a specific override for this particular cluster, therefore I'll add a new file in this folder, called `keycloak.yaml`, with the specific override.

```yaml
spec:
  values:
  ...
    database:
      vendor: dev-file # https://www.keycloak.org/server/db
    args:
      - start-dev # https://www.keycloak.org/server/configuration#_starting_keycloak_in_development_mode
    extraEnv: |
      - name: KC_HOSTNAME
        value: http://keycloak.localhost:8080
      - name: KC_LOG_LEVEL
        value: INFO
    http:
      relativePath: "/"
    # https://www.keycloak.org/server/configuration#_creating_the_initial_admin_user
    # Created manually
    extraEnvFrom: |
      - secretRef:
          name: keycloak-secrets
    # De facto protocol to add client data in headers
    proxy:
      mode: xforwarded
    ingress:
      enabled: true
      ingressClassName: traefik
      rules:
        - host: keycloak.localhost
          paths:
            - path: '{{ tpl .Values.http.relativePath $ | trimSuffix "/" }}/'
              pathType: Prefix
```

> [!WARNING]
> The k8s secret called `keycloak-secrets` is not in the github repository. It was created
> manually using the following command (could be saved in Vault and use ESO to pull the
> secret for better security):

```bash
kubectl create secret generic keycloak-secrets -n keycloak \
  --from-literal=KC_BOOTSTRAP_ADMIN_USERNAME='<ADMIN_USERNAME>' \
  --from-literal=KC_BOOTSTRAP_ADMIN_PASSWORD='<ADMIN_PASSWORD>'
```

In `/infrastructure/environment/dev` I do not have to add override values, and the `kustomization.yaml` from that folder points directly to `../../controllers`.

The version of the chart is written in `/clusters/tc-dev-euc1/cluster-vars.yaml` and used in `/infrastructure/controllers/keycloak/helmrelease.yaml`.

After installation, check the keycloak pods:

```bash
kubectl get pods -n keycloak
NAME                   READY   STATUS    RESTARTS       AGE
keycloak-keycloakx-0   1/1     Running   1 (2d3h ago)   2d20h
```

Open the browser and type: `http://keycloak.localhost:8080/`. The username and the password are the ones set in `keycloak-secrets` above.

### Create a new realm

Click on `Manage realms`, `Create realm`, give it a name (`tc-loadtester`) and click on `Create`.

### Create a user in that realm

Make sure that you are on that specific realm (`tc-loadtester`).
Click on `Users`, `Add user`, switch `Email verified` to `On`, and fill the following: `Username: tc-loadtester-1`, `Email: tc-loadtester-1@load.com`, `First name: tc-loadtester-1`, `Last name: tc-loadtester-1`. Click on `Credentials`, `Set password`, set a password of your choice and make sure that `Temporary` is `Off`.

### Create a client in that realm

Make sure you are on the `tc-loadtester` realm. Click on `Clients`, `Create client`.

**General settings:**

- `Client type`: `OpenID Connect`
- `Client ID`: `tc-loadtester-1`

Click `Next`.

**Capability config:**

- `Client authentication`: `On` (confidential client — oauth2-proxy authenticates with a client secret)
- `Authorization`: `Off`
- `Standard flow`: `On`
- `Direct access grants`: `On` (lets the load test obtain a token with `grant_type=password`, without a browser)
- the rest: `Off`

Click `Next`.

**Login settings:**

- `Valid redirect URIs`: `http://loadtester.localhost:8080/oauth2/callback` (must match oauth2-proxy's `--redirect-url`)
- `Valid post logout redirect URIs`: `http://loadtester.localhost:8080/*` (optional)
- `Web origins`: `http://loadtester.localhost:8080` (optional)

Click `Save`.

Open the `Credentials` tab of the client and copy the `Client secret` — this is the value oauth2-proxy needs.

> [!WARNING]
> Like `keycloak-secrets`, the `oauth2-proxy` secret is not in the repository and was created
> manually. It holds the client id, the client secret copied above, and a random cookie
> secret (`cookie-secret` must be 16, 24, or 32 bytes):

```bash
kubectl create secret generic oauth2-proxy -n tc-loadtester \
  --from-literal=client-id='tc-loadtester-1' \
  --from-literal=client-secret='<CLIENT_SECRET_FROM_KEYCLOAK>' \
  --from-literal=cookie-secret="$(openssl rand -base64 32)"
```

The oauth2-proxy deployment reads these three keys via `secretKeyRef` (`OAUTH2_PROXY_CLIENT_ID` / `OAUTH2_PROXY_CLIENT_SECRET` / `OAUTH2_PROXY_COOKIE_SECRET`).

## Why we need oauth2-proxy

> [!NOTE]
> Our application has an Ingress for `/`. In order to integrate the oauth2-proxy in this flow,
> traefik controller has a resource called `Middleware` that can be found
> [here](https://github.com/stefanescualexandrumihai/TechnicalChallenge-loadtester/blob/main/base/middleware.yaml),
> which creates a subrequest for oauth2-proxy when we access `loadtester.localhost`.

```yaml
kind: Middleware
spec:
  forwardAuth:
    address: http://oauth2-proxy.tc-loadtester.svc.cluster.local/
```

The ingress itself must be annotated with:

```yaml
  annotations:
    traefik.ingress.kubernetes.io/router.middlewares: tc-loadtester-oidc-auth@kubernetescrd
```

The oauth2-proxy needs the login URL for keycloak.

```bash
--login-url=http://keycloak.localhost:8080/realms/tc-loadtester/protocol/openid-connect/auth
```

After the oauth2, keycloak needs a redirect URL too.

```bash
--redirect-url=http://loadtester.localhost:8080/oauth2/callback
```

- `--redeem-url` - is used for token exchange.
- `--oidc-jwks-url` - is where oauth2-proxy fetches Keycloak's public keys to verify token signatures.

For testing, I set the `--skip-jwt-bearer-tokens=true`. If a request has a valid `Authorization: Bearer ...`, it is accepted.

- `--skip-oidc-discovery=true` — manual discovery via the above urls
- `--oidc-issuer-url` — issuer url, to check `iss` from tokens
- `--cookie-secure=false` — we use HTTP, not HTTPS
- `--reverse-proxy=true` — tells oauth2-proxy that it is behind a reverse proxy server (traefik)
- `--set-xauthrequest=true` — makes oauth2-proxy return the identity headers listed in `authResponseHeaders`, which Traefik copies onto the request so the app knows who the user is
- `--skip-provider-button=true` — skip the intermediary page `Sign in with Keycloak`
- `--email-domain=*` — matches all domains

After keycloak authenticates the user, it will respond with 302 in order to redirect to the oauth2-proxy `/callback` endpoint, setting the authorization code. Oauth2-proxy will call the `redeem` endpoint to obtain a token, calling the keycloak endpoint within the cluster. The oauth2-proxy encrypts the user's metadata with the cookie secret and passes it to the user (it will be the ticket for the next request that will arrive in oauth2-proxy, which will see that the user is already authenticated).

> [!IMPORTANT]
> Browser → Traefik → subrequest → oauth2-proxy: "no cookie" → 401 → browser is redirected to
> Keycloak → login → 302 with code to /callback → browser carries the code → Traefik
> (Ingress /oauth2, without middleware) → oauth2-proxy → POST pod-to-pod with code+secret →
> token → Set-Cookie + redirect to the original URL → browser loads the page again, with
> cookie → subrequest → 200 → tc-loadtester.

Let's test!

Open the browser and type `http://loadtester.localhost:8080/`. You should be redirected to keycloak. Enter the user and password and voila!