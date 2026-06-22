# AzureX Demo

A self-contained, single-page static website with a retro **hacker-terminal** aesthetic — Matrix rain background, an animated CRT terminal that replays a fake pentest session, live "system" stats, rotating security quotes, and a real-time clock. Everything lives in one file: [`index.html`](./index.html) (HTML, CSS, and vanilla JavaScript — no build step, no dependencies).

## Preview locally

Because it's a single static file, you can just open it directly:

```bash
open index.html        # macOS
```

Or serve it over HTTP (recommended, mirrors how it runs in production):

```bash
# Python 3
python3 -m http.server 8080
# then visit http://localhost:8080
```

```bash
# Node (if you have it)
npx serve .
```

## Deploy to Azure Static Web Apps

[Azure Static Web Apps](https://learn.microsoft.com/azure/static-web-apps/) hosts this site for free (Free tier) and wires up CI/CD from GitHub automatically. Pick **one** of the two options below.

### Prerequisites

- An [Azure account](https://azure.microsoft.com/free/) (the Free tier covers this app).
- This repo pushed to GitHub: `https://github.com/dtdrupasinghe/azurex-demo`.

### Option A — Azure Portal (no CLI, recommended for first deploy)

1. Sign in to the [Azure Portal](https://portal.azure.com).
2. Click **Create a resource** → search for **Static Web App** → **Create**.
3. Fill in the **Basics** tab:
   - **Subscription**: your subscription.
   - **Resource Group**: create a new one, e.g. `azurex-demo-rg`.
   - **Name**: `azurex-demo`.
   - **Plan type**: **Free**.
   - **Region**: pick the one closest to you.
4. Under **Deployment details**:
   - **Source**: **GitHub** → sign in and authorize Azure.
   - **Organization**: `dtdrupasinghe`
   - **Repository**: `azurex-demo`
   - **Branch**: `main`
5. Under **Build Details**:
   - **Build Presets**: **Custom**.
   - **App location**: `/`
   - **Api location**: *(leave blank)*
   - **Output location**: *(leave blank — the site is served straight from the repo root)*
6. Click **Review + create** → **Create**.

Azure commits a workflow file (`.github/workflows/azure-static-web-apps-*.yml`) to your repo and runs it. When the GitHub Actions run finishes, your site is live at the URL shown on the Static Web App's **Overview** page (e.g. `https://<random-name>.azurestaticapps.net`).

### Option B — Azure CLI

Requires the [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) with the Static Web Apps extension.

```bash
# 1. Log in
az login

# 2. Create a resource group
az group create --name azurex-demo-rg --location eastus

# 3. Create the Static Web App linked to GitHub.
#    This opens a browser to authorize GitHub and commits the
#    deployment workflow to your repo automatically.
az staticwebapp create \
  --name azurex-demo \
  --resource-group azurex-demo-rg \
  --source https://github.com/dtdrupasinghe/azurex-demo \
  --branch main \
  --app-location "/" \
  --output-location "" \
  --login-with-github
```

After the command completes, get the live URL:

```bash
az staticwebapp show \
  --name azurex-demo \
  --resource-group azurex-demo-rg \
  --query "defaultHostname" -o tsv
```

## How CI/CD works after setup

Once the Static Web App is connected, a GitHub Actions workflow lives in `.github/workflows/`. **Every push to `main` automatically rebuilds and redeploys** the site. Pull requests get their own temporary preview URL until merged or closed.

## Configuration notes

- **App location** is `/` because `index.html` sits at the repo root.
- **Output location** is empty because there is no build step — Azure serves the files as-is.
- To add routing rules, custom headers, or fallback routes, drop a [`staticwebapp.config.json`](https://learn.microsoft.com/azure/static-web-apps/configuration) at the repo root.

## Cleanup

To remove all Azure resources created above:

```bash
az group delete --name azurex-demo-rg --yes --no-wait
```
