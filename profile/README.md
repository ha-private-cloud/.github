# ClusterKeep

ClusterKeep is my high availability private cloud storage project. Most people's files are held by a small handful of large US companies, which puts them under US law wherever the servers are. ClusterKeep lets someone host their own cloud storage at home, reachable from their home network or anywhere in the world, without needing to be an infrastructure expert to run it.

## Disclaimer

The clusterkeep URL will often be unavailable as I currently only turn the machines on when I'm developing or using the firewall VM, this is a WIP project and public access won't be offered. 

The GitHub organisation profile will be updated periodically as progress is made in the project and the end goal is to have a complete ecosystem within my project deadlines and a runbook which will allow others to deploy the ClusterKeep platform themselves.

## What's running

Three Lenovo ThinkCentre computers run Proxmox VE. Each one hosts a Talos Linux control plane and two workers, so the Kubernetes cluster keeps running if any one computer goes down. Two of them also host an NFS server for storage, and a CI bastion VM runs the GitHub Actions runners and internal DNS. An OPNsense VM on a fourth computer is the edge firewall.

People join with an invite code and log in on the main site, then upload, organise, view and download their files in the storage site. Admins manage accounts and cluster access in an admin console behind Cloudflare Access, and auth-api handles every login.

Production is public through a Cloudflare Tunnel, so nothing on the home network is opened to the internet.

Every user's files are encrypted with their own key, locked by their password and a recovery code only they hold.

## Repositories

### Apps

| Repo | What it is |
|---|---|
| [clusterkeep-ui](https://github.com/ha-private-cloud/clusterkeep-ui) | Front page, joining, logging in and storage recovery. Flask |
| [storage-ui](https://github.com/ha-private-cloud/storage-ui) | The file browser. Flask and vanilla JavaScript |
| [storage-api](https://github.com/ha-private-cloud/storage-api) | Encrypted per-user storage, only reachable inside the cluster. FastAPI |
| [auth-api](https://github.com/ha-private-cloud/auth-api) | Accounts, sessions and OpenID Connect. FastAPI |
| [admin-ui](https://github.com/ha-private-cloud/admin-ui) | Admin console for accounts and cluster access. Flask |

### Infrastructure

| Repo | What it is |
|---|---|
| [proxmox-tofu](https://github.com/ha-private-cloud/proxmox-tofu) | The VMs on the Proxmox hosts and the OPNsense firewall |
| [cluster-config](https://github.com/ha-private-cloud/cluster-config) | Shared cluster services: ingress, NFS storage, Headlamp, the Nexus registry, the Cloudflare Tunnel and namespaces |
| [cluster-auth](https://github.com/ha-private-cloud/cluster-auth) | Which auth-api groups get which level of cluster access |
| [cluster-network](https://github.com/ha-private-cloud/cluster-network) | DNS, static addresses, certificates and recovery runbooks |
| [cluster-ci](https://github.com/ha-private-cloud/cluster-ci) | Runs the tests and builds images inside the cluster |
| [cluster-cli](https://github.com/ha-private-cloud/cluster-cli) | The command line tool behind every build and deploy |
| [pipelines](https://github.com/ha-private-cloud/pipelines) | Shared GitHub Actions workflows |
| [helm-charts](https://github.com/ha-private-cloud/helm-charts) | Helm charts for the apps |
| [dev-env-config](https://github.com/ha-private-cloud/dev-env-config) | Which apps common-dev-env loads when running everything locally |
| [cluster-rbac](https://github.com/ha-private-cloud/cluster-rbac) | Retired, replaced by cluster-auth |

## How changes reach production

Every app has `development`, `preview` and `main` branches. Pushing to `development` or `preview` deploys to that environment automatically. `preview` and `main` only take changes through pull requests. Releases are `vX.Y.Z` tags on `main`, and deploying one to production is a manual step.

The infrastructure repos plan every tagged change automatically, and nothing is applied without me running it by hand.

## Running it locally

All five apps run together on one machine with HM Land Registry's [common-dev-env](https://github.com/LandRegistry/common-dev-env). The [dev-env-config](https://github.com/ha-private-cloud/dev-env-config) repository tells it which apps to load.

You need Docker with Docker Compose, Git and Ruby 3.0 or newer.

```sh
git clone https://github.com/LandRegistry/common-dev-env.git
cd common-dev-env
source run.sh up
```

The first time, it asks for the URL of a configuration repository, which is `https://github.com/ha-private-cloud/dev-env-config.git`.

It clones every app into `apps/` on its `development` branch, then builds and starts them. When it finishes, go to http://localhost:9080/login and log in as `admin` with the password `dev-password-please-change`. You'll be asked to choose a new password and then shown your storage recovery code.

| App | Local address |
|---|---|
| clusterkeep-ui | http://localhost:9080 |
| storage-ui | http://localhost:9081 |
| auth-api | http://localhost:9082/docs |
| admin-ui | http://localhost:9083 |
| storage-api | http://localhost:9084/docs |

Make changes inside each app's folder under `apps/`. Every `up` or `reload` pulls the latest `development`, but an app is left alone while you have a different branch checked out.

`source run.sh halt` stops everything, `source run.sh reload` rebuilds and restarts it, and `source run.sh destroy` removes it all.

## Built with

Proxmox VE, Talos Linux, Kubernetes, OpenTofu, Helm, Python with Flask and FastAPI, Postgres, Redis, Tailwind CSS, GitHub Actions, Kaniko, Nexus and Cloudflare.

The project plan and progress are on the [Product Backlog](https://github.com/orgs/ha-private-cloud/projects/2) board.
