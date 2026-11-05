# LegalLens AKS deployment

Pushing to `main` runs the root `.github/workflows/ci.yml`. After the frontend build and backend checks pass, CI publishes both containers to `legallensacs.azurecr.io` using the full Git commit SHA as the tag. `.github/workflows/cd.yml` then deploys those exact images to `aks-legallens` in `rg-legallens`.

The Kubernetes manifests use a public Azure LoadBalancer for the frontend and an internal service for the API. Nginx serves the frontend and proxies `/api`, `/health`, and `/ready` to the backend. The backend uses one replica and an 8 GiB Azure managed disk claim for its SQLite database and uploaded documents.

## One-time GitHub setup

Create these repository actions secrets in `chandipriyanaragam/LegalLens`:

| Secret | Value |
|---|---|
| `AZURE_CLIENT_ID` | Client ID of an Entra application or user-assigned identity configured for GitHub OIDC |
| `AZURE_TENANT_ID` | Azure tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Subscription containing `rg-legallens` |
| `OPENAI_API_KEY` | API key used by the backend |

Configure an OIDC federated credential for issuer `https://token.actions.githubusercontent.com`, audience `api://AzureADTokenExchange`, and subject `repo:chandipriyanaragam/LegalLens:ref:refs/heads/main`. The Azure identity needs `AcrPush` on `legallensacs` and `Azure Kubernetes Service Cluster User Role` on `aks-legallens`. If the cluster uses Azure RBAC for Kubernetes authorization, also grant the identity `Azure Kubernetes Service RBAC Writer` scoped to that cluster.

The AKS node identity has `AcrPull` on the registry. The CD workflow creates/updates the `legallens-secrets` Kubernetes secret from `OPENAI_API_KEY` on each deployment.

After the first successful deployment, the workflow summary reports the frontend's public IP address. A custom domain and TLS certificate can be added separately.
