# Harness GitOps: Multi-service DEV demo

This repo deploys three independent demo HTTP services to a Kubernetes namespace named `dev` using one Kustomize-based Argo CD / Harness GitOps Application.

Services:
- `orders` (ClusterIP service on port 80)
- `payments` (ClusterIP service on port 80)
- `billing` (ClusterIP service on port 80)

Each service runs NGINX and returns a different HTML page. This is a lightweight deployment demo, not business-logic implementations.

## Repository path

Configure the Harness GitOps Application source path to `k8s/dev`, target revision `main`, and destination namespace `dev`.

## Files

- `k8s/dev/kustomization.yaml`: deploys all resources together.
- `k8s/dev/namespace.yaml`: creates namespace `dev`.
- `k8s/dev/orders.yaml`: ConfigMap, Deployment, and Service for orders.
- `k8s/dev/payments.yaml`: ConfigMap, Deployment, and Service for payments.
- `k8s/dev/billing.yaml`: ConfigMap, Deployment, and Service for billing.

## Push to GitHub

Create an empty GitHub repository, for example `harness-gitops-multi-service-dev`. From this directory run:

```bash
git init
git add .
git commit -m "Add multi-service dev GitOps demo"
git branch -M main
git remote add origin https://github.com/<YOUR_GITHUB_USERNAME>/harness-gitops-multi-service-dev.git
git push -u origin main
```

Replace `<YOUR_GITHUB_USERNAME>` with your GitHub username. For a private repository, configure credentials in Harness GitOps Repository settings; for a public repository, anonymous read access may be used.

## Create the Harness GitOps entities

1. Under your Harness project, go to **GitOps → Settings → Repositories → New Repository**. Choose Git, select your healthy GitOps Agent, and enter the repository URL. Configure credentials if private, then verify and finish.
2. Under **GitOps → Settings → Clusters → New Cluster**, select the same Agent and choose to use that Agent's credentials. Verify and finish. Give it a clear name such as `rancher-dev`.
3. Under **GitOps → Applications → New Application**, name it `multi-service-dev`, choose Argo and the same Agent.
4. Configure the source repository you just registered, target revision `main`, and path `k8s/dev`.
5. Choose the `rancher-dev` cluster and destination namespace `dev`. Use manual sync for the first test, and enable Auto-Create Namespace if the wizard offers that option.
6. Finish, open the application, and click **Sync → Synchronize**.

## Verify from your Mac

```bash
kubectl get deployments,services,pods -n dev
kubectl get applications -n gitops-agent
```

Expected Deployments: `orders`, `payments`, `billing`; each should become ready with one replica.

To test a service, run a port-forward in one terminal:

```bash
kubectl port-forward -n dev svc/orders 8081:80
```

In another terminal:

```bash
curl http://localhost:8081
```

Repeat for payments (`svc/payments`, port `8082:80`) and billing (`svc/billing`, port `8083:80`). Stop each port-forward with Ctrl+C.

To deploy an update, edit a manifest and push a commit to `main`, then refresh/sync the Harness GitOps Application.
