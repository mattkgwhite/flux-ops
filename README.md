<div align="center">

<img src="https://camo.githubusercontent.com/5b298bf6b0596795602bd771c5bddbb963e83e0f/68747470733a2f2f692e696d6775722e636f6d2f7031527a586a512e706e67" align="center" width="144px" height="144px"/>

### My flux operations repository :octocat:

_... managed with Flux, Renovate and GitHub Actions_ 🤖

</div>

<div align="center">

[![Discord](https://img.shields.io/discord/673534664354430999?style=for-the-badge&label&logo=discord&logoColor=white&color=blue)](https://discord.gg/k8s-at-home)
[![Kubernetes](https://img.shields.io/badge/v1.26-blue?style=for-the-badge&logo=kubernetes&logoColor=white)](https://talos.dev/)
[![Renovate](https://img.shields.io/github/actions/workflow/status/coolguy1771/home-ops/renovate.yaml?branch=main&label=&logo=renovatebot&style=for-the-badge&color=blue)](https://github.com/coolguy1771/home-ops/actions/workflows/renovate.yaml)

</div>

---

## 📖 Overview

This is a mono repository for my home infrastructure and Kubernetes cluster. I try to adhere to Infrastructure as Code (IaC) and GitOps practices using the tools like [Kubernetes](https://kubernetes.io/), [Flux](https://github.com/fluxcd/flux2), [Renovate](https://github.com/renovatebot/renovate) and [GitHub Actions](https://github.com/features/actions).

---

## 📝 Prerequisites

1. You **MUST** have a Domain or Domains, which in this case is managed by [AWS Route53](https://aws.amazon.com/route53/)
  - There are alternative DNS providers available, which are shown [here](https://github.com/kubernetes-sigs/external-dns/#deploying-to-a-cluster)
2. Secrets will be commited to your Git repository **AND** they will be encrypted by SOPS.
3. In order for this all to work you have to use nodes that have access to the internet.
4. Only **amd64** and/or **arm64** nodes are supported.

### 💻 Systems

- A single VPS / VM that can access the internet via a single public IP
  - The ability to scale this node. **Note:** *There will be a cost implication with scaling the node, and this depends entirely on workload for the Kubernetes deployment itself*
  The VPS / VM can be ARM64/AMD64 bare metal if required.
- An understand of Amazon Web Services and how to configure IAM Roles for permissions to specific Hosted Zones and the Hosted Zones specific IDs.

## ⛵ Kubernetes

There is a template over at [onedr0p/flux-cluster-template](https://github.com/onedr0p/flux-cluster-template) if you wanted to try and follow along with some of the practices I use here.

### Installation

My cluster is [k3s](https://k3s.io/) provisioned overtop bare-metal Ubuntu Server. This is a semi hyper-converged cluster using one node which is a VPS provided by [Hetzner](https://hetzner.com)'s Cloud offering, workloads and block storage using the [HCloud CSI Driver](https://github.com/hetznercloud/csi-driver/) to access Volumes that can be mounted directly to the VPS or via a [Hetzner Storage Box](https://www.hetzner.com/storage/storage-box).

### Core Components

- [cert-manager](https://cert-manager.io/docs/): Creates SSL certificates for services in my Kubernetes cluster.
- [external-dns](https://github.com/kubernetes-sigs/external-dns): Automatically manages DNS records from my cluster in a cloud DNS provider.
- [external-secrets](https://github.com/external-secrets/external-secrets/): Managed Kubernetes secrets using [1Password Connect](https://github.com/1Password/connect).
- [ingress-nginx](https://github.com/kubernetes/ingress-nginx/): Ingress controller to expose HTTP traffic to pods over DNS.
- [sops](https://toolkit.fluxcd.io/guides/mozilla-sops/): Managed secrets for Kubernetes, Ansible and Terraform which are commited to Git.
- [volsync](https://github.com/backube/volsync) and [snapscheduler](https://github.com/backube/snapscheduler): Backup and recovery of persistent volume claims.

### GitOps

[Flux](https://github.com/fluxcd/flux2) watches my [kubernetes](./kubernetes/) folder (see Directories below) and makes the changes to my cluster based on the YAML manifests.

The way Flux works for me here is it will recursively search the [kubernetes/apps](./kubernetes/apps) folder until it finds the most top level `kustomization.yaml` per directory and then apply all the resources listed in it. That aforementioned `kustomization.yaml` will generally only have a namespace resource and one or many Flux kustomizations. Those Flux kustomizations will generally have a `HelmRelease` or other resources related to the application underneath it which will be applied.

[Renovate](https://github.com/renovatebot/renovate) watches my **entire** repository looking for dependency updates, when they are found a PR is automatically created. When some PRs are merged [Flux](https://github.com/fluxcd/flux2) applies the changes to my cluster.

### Directories

This Git repository contains the following directories under [kubernetes](./kubernetes/).

```sh
📁 kubernetes      # Kubernetes cluster defined as code
├─📁 bootstrap     # Flux installation
├─📁 flux          # Main Flux configuration of repository
└─📁 apps          # Apps deployed into my cluster grouped by namespace (see below)
```

## 🔧 Workstation Tools

📍 Install the **most recent version** of the CLI tools below. If you are **having trouble with future steps**, it is very likely you don't have the most recent version of these CLI tools, **!especially sops AND yq!**.

1. Install the following CLI tools on your workstation, if you are **NOT** using [Homebrew](https://brew.sh/) on MacOS or Linux **ignore** steps 4 and 5.

   - Required: [age](https://github.com/FiloSottile/age), [ansible](https://www.ansible.com), [flux](https://toolkit.fluxcd.io/), [weave-gitops](https://docs.gitops.weave.works/docs/installation/weave-gitops/), [cloudflared](https://github.com/cloudflare/cloudflared), [cilium-cli](https://github.com/cilium/cilium-cli), [go-task](https://github.com/go-task/task), [direnv](https://github.com/direnv/direnv), [ipcalc](http://jodies.de/ipcalc), [jq](https://stedolan.github.io/jq/), [kubectl](https://kubernetes.io/docs/tasks/tools/), [python-pip3](https://pypi.org/project/pip/), [pre-commit](https://github.com/pre-commit/pre-commit), [sops v3](https://github.com/mozilla/sops), [yq v4](https://github.com/mikefarah/yq)

   - Recommended: [helm](https://helm.sh/), [kustomize](https://github.com/kubernetes-sigs/kustomize), [stern](https://github.com/stern/stern), [yamllint](https://github.com/adrienverge/yamllint)

2. This guide heavily relies on [go-task](https://github.com/go-task/task) as a framework for setting things up. It is advised to learn and understand the commands it is running under the hood.

3. Install Python 3 and pip3 using your Linux OS package manager, or Homebrew if using MacOS.

   - Ensure `pip3` is working on your command line by running `pip3 --version`

4. [Homebrew] Install [go-task](https://github.com/go-task/task)

   ```sh
   brew install go-task/tap/go-task
   ```

5. [Homebrew] Install workstation dependencies

   ```sh
   task init
   ```

### 🔐 Setting up Age

📍 Here we will create a Age Private and Public key. Using [SOPS](https://github.com/mozilla/sops) with [Age](https://github.com/FiloSottile/age) allows us to encrypt secrets and use them in Ansible and Flux.

1. Create a Age Private / Public Key

   ```sh
   age-keygen -o age.agekey
   ```

2. Set up the directory for the Age key and move the Age file to it

   ```sh
   mkdir -p ~/.config/sops/age
   mv age.agekey ~/.config/sops/age/keys.txt
   ```

3. Export the `SOPS_AGE_KEY_FILE` variable in your `bashrc`, `zshrc` or `config.fish` and source it, e.g.

   ```sh
   export SOPS_AGE_KEY_FILE=~/.config/sops/age/keys.txt
   source ~/.bashrc
   ```

### 📄 Configuration

📍 The `.config.env` file contains necessary configuration that is needed by Ansible and Flux.

1. Copy the `.config.sample.env` to `.config.env` and start filling out all the environment variables.

   **All are required** unless otherwise noted in the comments.

   ```sh
   cp .config.sample.env .config.env
   ```

2. Once that is done, verify the configuration is correct by running:

   ```sh
   task verify
   ```

3. If you do not encounter any errors run start having the script wire up the templated files and place them where they need to be.

   ```sh
   task configure
   ```

⚠️ This will print out the clear-text passwords for Grafana and Weave Gitops if you had them set to `generated` in your `.config.env`. Take note of these, you'll need them to log into the applications.

---

### 🔹 GitOps with Flux

📍 Here we will be installing [flux](https://toolkit.fluxcd.io/) after some quick bootstrap steps.

1. Verify Flux can be installed

   ```sh
   task cluster:verify
   # ► checking prerequisites
   # ✔ kubectl 1.21.5 >=1.18.0-0
   # ✔ Kubernetes 1.21.5+k3s1 >=1.16.0-0
   # ✔ prerequisites checks passed
   ```

2. Push you changes to git

   📍 **Verify** all the `*.sops.yaml` and `*.sops.yml` files under the `./ansible`, and `./kubernetes` directories are **encrypted** with SOPS

   ```sh
   git add -A
   git commit -m "Initial commit :rocket:"
   git push
   ```

3. Install Flux and sync the cluster to the Git repository

   ```sh
   task cluster:install
   # namespace/flux-system configured
   # customresourcedefinition.apiextensions.k8s.io/alerts.notification.toolkit.fluxcd.io created
   ```

4. Verify Flux components are running in the cluster

   ```sh
   task cluster:pods -- -n flux-system
   # NAME                                       READY   STATUS    RESTARTS   AGE
   # helm-controller-5bbd94c75-89sb4            1/1     Running   0          1h
   # kustomize-controller-7b67b6b77d-nqc67      1/1     Running   0          1h
   # notification-controller-7c46575844-k4bvr   1/1     Running   0          1h
   # source-controller-7d6875bcb4-zqw9f         1/1     Running   0          1h
   ```

### 🎤 Verification Steps

_Mic check, 1, 2_ - In a few moments applications should be lighting up like a Christmas tree 🎄

You are able to run all the commands below with one task

```sh
task cluster:resources
```

1. View the Flux Git Repositories

   ```sh
   task cluster:gitrepositories
   ```

2. View the Flux kustomizations

   ```sh
   task cluster:kustomizations
   ```

3. View all the Flux Helm Releases

   ```sh
   task cluster:helmreleases
   ```

4. View all the Flux Helm Repositories

   ```sh
   task cluster:helmrepositories
   ```

5. View all the Pods

   ```sh
   task cluster:pods
   ```

6. View all the certificates and certificate requests

   ```sh
   task cluster:certificates
   ```

7. View all the ingresses

   ```sh
   task cluster:ingresses
   ```

🏆 **Congratulations** if all goes smooth you'll have a Kubernetes cluster managed by Flux, your Git repository is driving the state of your cluster.

☢️ If you run into problems, you can run `task ansible:nuke` to destroy the k3s cluster and start over.

🧠 Now it's time to pause and go get some coffee ☕ because next is describing how DNS is handled.

## 📣 Post installation

### 🌱 Environment

[direnv](https://direnv.net/) will make it so anytime you `cd` to your repo's directory it export the required environment variables (e.g. `KUBECONFIG`). To set this up make sure you [hook it into your shell](https://direnv.net/docs/hook.html) and after that is done, run `direnv allow` while in your repos directory.

### 🌐 DNS

📍 The `external-dns` application created in the `networking` namespace will handle creating public DNS records. By default, `echo-server` and the `flux-webhook` are the only public sub-domains exposed. In order to make additional applications public you must set an ingress annotation (`external-dns.alpha.kubernetes.io/target`) like done in the `HelmRelease` for `echo-server`.

### 🤖 Renovatebot

[Renovatebot](https://www.mend.io/free-developer-tools/renovate/) will scan your repository and offer PRs when it finds dependencies out of date. Common dependencies it will discover and update are Flux, Ansible Galaxy Roles, Terraform Providers, Kubernetes Helm Charts, Kubernetes Container Images, Pre-commit hooks updates, and more!

The base Renovate configuration provided in your repository can be view at [.github/renovate.json5](https://github.com/onedr0p/flux-cluster-template/blob/main/.github/renovate.json5). If you notice this only runs on weekends and you can [change the schedule to anything you want](https://docs.renovatebot.com/presets-schedule/) or simply remove it.

To enable Renovate on your repository, click the 'Configure' button over at their [Github app page](https://github.com/apps/renovate) and choose your repository. Over time Renovate will create PRs for out-of-date dependencies it finds. Any merged PRs that are in the kubernetes directory Flux will deploy.

### 🪝 Github Webhook

Flux is pull-based by design meaning it will periodically check your git repository for changes, using a webhook you can enable Flux to update your cluster on `git push`. In order to configure Github to send `push` events from your repository to the Flux webhook receiver you will need two things:

1. Webhook URL - Your webhook receiver will be deployed on `https://flux-webhook.${BOOTSTRAP_CLOUDFLARE_DOMAIN}/hook/:hookId`. In order to find out your hook id you can run the following command:

   ```sh
   kubectl -n flux-system get receiver/github-receiver
   # NAME              AGE    READY   STATUS
   # github-receiver   6h8m   True    Receiver initialized with URL: /hook/12ebd1e363c641dc3c2e430ecf3cee2b3c7a5ac9e1234506f6f5f3ce1230e123
   ```

   So if my domain was `onedr0p.com` the full url would look like this:

   ```text
   https://flux-webhook.onedr0p.com/hook/12ebd1e363c641dc3c2e430ecf3cee2b3c7a5ac9e1234506f6f5f3ce1230e123
   ```

2. Webhook secret - Your webhook secret can be found by decrypting the `secret.sops.yaml` using the following command:

   ```sh
   sops -d ./kubernetes/apps/flux-system/addons/webhooks/github/secret.sops.yaml | yq .stringData.token
   ```

   **Note:** Don't forget to update the `BOOTSTRAP_FLUX_GITHUB_WEBHOOK_SECRET` variable in your `.config.env` file so it matches the generated secret if applicable

Now that you have the webhook url and secret, it's time to set everything up on the Github repository side. Navigate to the settings of your repository on Github, under "Settings/Webhooks" press the "Add webhook" button. Fill in the webhook url and your secret.

### 💾 Storage

Rancher's `local-path-provisioner` is a great start for storage but soon you might find you need more features like replicated block storage, or to connect to a NFS/SMB/iSCSI server. Check out the projects below to read up more on some storage solutions that might work for you.

- [rook-ceph](https://github.com/rook/rook)
- [longhorn](https://github.com/longhorn/longhorn)
- [openebs](https://github.com/openebs/openebs)
- [nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)
- [democratic-csi](https://github.com/democratic-csi/democratic-csi)
- [csi-driver-nfs](https://github.com/kubernetes-csi/csi-driver-nfs)
- [synology-csi](https://github.com/SynologyOpenSource/synology-csi)

### 🔏 Authenticate Flux over SSH

Authenticating Flux to your git repository has a couple benefits like using a private git repository and/or using the Flux [Image Automation Controllers](https://fluxcd.io/docs/components/image/).

By default this template only works on a public GitHub repository, it is advised to keep your repository public.

The benefits of a public repository include:

- Debugging or asking for help, you can provide a link to a resource you are having issues with.
- Adding a topic to your repository of `k8s-at-home` to be included in the [k8s-at-home-search](https://whazor.github.io/k8s-at-home-search/). This search helps people discover different configurations of Helm charts across others Flux based repositories.

<details>
  <summary>Expand to read guide on adding Flux SSH authentication</summary>

1. Generate new SSH key:
   ```sh
   ssh-keygen -t ecdsa -b 521 -C "github-deploy-key" -f ./kubernetes/bootstrap/github-deploy.key -q -P ""
   ```
2. Paste public key in the deploy keys section of your repository settings
3. Create sops secret in `./kubernetes/bootstrap/github-deploy-key.sops.yaml` with the contents of:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: github-deploy-key
     namespace: flux-system
   stringData:
     # 3a. Contents of github-deploy-key
     identity: |
       -----BEGIN OPENSSH PRIVATE KEY-----
           ...
       -----END OPENSSH PRIVATE KEY-----
     # 3b. Output of curl --silent https://api.github.com/meta | jq --raw-output '"github.com "+.ssh_keys[]'
     known_hosts: |
       github.com ssh-ed25519 ...
       github.com ecdsa-sha2-nistp256 ...
       github.com ssh-rsa ...
   ```
4. Encrypt secret:
   ```sh
   sops --encrypt --in-place ./kubernetes/bootstrap/github-deploy-key.sops.yaml
   ```
5. Apply secret to cluster:
   ```sh
   sops --decrypt ./kubernetes/bootstrap/github-deploy-key.sops.yaml | kubectl apply -f -
   ```
6. Update `./kubernetes/flux/config/cluster.yaml`:
   ```yaml
   apiVersion: source.toolkit.fluxcd.io/v1beta2
   kind: GitRepository
   metadata:
     name: home-kubernetes
     namespace: flux-system
   spec:
     interval: 10m
     # 6a: Change this to your user and repo names
     url: ssh://git@github.com/$user/$repo
     ref:
       branch: main
     secretRef:
       name: github-deploy-key
   ```
7. Commit and push changes
8. Force flux to reconcile your changes
   ```sh
   task cluster:reconcile
   ```
9. Verify git repository is now using SSH:
   ```sh
   task cluster:gitrepositories
   ```
10. Optionally set your repository to Private in your repository settings.
</details>

### 💨 Kubernetes Dashboard

Included in your cluster is the [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/). Inorder to log into this you will have to get the secret token from the cluster using the command below.

```sh
kubectl -n monitoring get secret kubernetes-dashboard -o jsonpath='{.data.token}' | base64 -d
```

You should be able to access the dashboard at `https://kubernetes.${SECRET_DOMAIN}`

## 🐛 Debugging

Below is a general guide on trying to debug an issue with an resource or application. For example, if a workload/resource is not showing up or a pod has started but in a `CrashLoopBackOff` or `Pending` state.

1. Start by checking all Flux Kustomizations & Git Repository & OCI Repository and verify they are healthy.

- `flux get sources oci -A`
- `flux get sources git -A`
- `flux get ks -A`

2. Then check all the Flux Helm Releases and verify they are healthy.

- `flux get hr -A`

3. Then check the if the pod is present.

- `kubectl -n <namespace> get pods`

4. Then check the logs of the pod if its there.

- `kubectl -n <namespace> logs <pod-name> -f`

Note: If a resource exists, running `kubectl -n <namespace> describe <resource> <name>` might give you insight into what the problem(s) could be.

Resolving problems that you have could take some tweaking of your YAML manifests in order to get things working, other times it could be a external factor like permissions on NFS. If you are unable to figure out your problem see the help section below.


## 🤝 Gratitude and Thanks

Thanks to all the people who donate their time to the [Kubernetes @Home](https://discord.gg/k8s-at-home) Discord community. A lot of inspiration for my cluster comes from the people that have shared their clusters using the [k8s-at-home](https://github.com/topics/k8s-at-home) GitHub topic. Be sure to check out the [Kubernetes @Home search](https://nanne.dev/k8s-at-home-search/) for ideas on how to deploy applications or get ideas on what you can deploy.

---

## 📜 Changelog

See my _awful_ [commit history](https://github.com/mattkgwhite/flux-ops/commits/main)

---

## 🔏 License

See [LICENSE](./LICENSE)