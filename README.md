# GitOps workflow for kubernetes cluster

![](https://i.imgur.com/qBbjyNx.png)

Leverage [WeaveWorks Flux](https://github.com/weaveworks/flux) to automate cluster state using code residing in this repo

## Setup

* See [cluster bootstrap instructions](setup/cluster/) for bootstrapping a kubernetes cluster
* See [gitops bootstrap instructions](setup/) for bootstrapping flux gitopsfor using this repo

## Workloads (by namespace)

### kube-system

See [kube-system](kube-system/) for details on system-level configurations (cert-manager, decsheduler, fluxcloud, intel gpu plugin, kured, metallb, nginx, sealed-secrets)

### default

See [default](default/) for details on regular workloads (frigate, home-assistant, hubot, minecraft, node-red, nzbget, plex, rabbitmq, radarr, rtorrent-flood, sonarr, unifi)

### Monitoring

See [monitoring](monitoring/) for details on regular workloads (chronograf, comcast usage, grafana, influxdb, cable modem stats, prometheus-operator, speedtest results, uptimerobot agent)

### Logs

See [logs](logs/) for details on logging solutions (loki, EFK Stack (elasticSearch, fluentd, kibana), elasticsearch-curator)

## Caveats

### New namespaces

If deploying a helm chart that needs to live in a new namespace, Flux seems to expect that the namespace is already created, or else the helm deployment will fail.  When deploying a helm chart in the traditional approach via the `helm` CLI, it would handle the namespace creation for you.  In Flx, you must explicitly create a helm chart (see [storage/rook/namespace.yaml](storage/rook/namespace.yaml) for an example of this)

### Deletions

[Flux doesn't handle deletions](https://github.com/weaveworks/flux/blob/master/site/faq.md#will-flux-delete-resources-that-are-no-longer-in-the-git-repository).  What this means is that if you remove something from the repo (or even change something to run in a different namespace), it will not clean-up the removed item.  This is a task that you must manually do.

To remove HelmRelease type entities from flux, you must manually delete the helmrelease object, e.g. to clean-up a helm release named `forwardauth`.  This should properly remove the helm chart and associated objects

```shell
kubectl -n kube-system delete helmrelease/forwardauth
```

### Secrets & Sensitive information

Secrets are handled via [vault](https://github.com/hashicorp/vault-helm) and the [vault-secrets-operator](https://github.com/ricoberger/vault-secrets-operator).

See [the kube-system readme](kube-system/README.md) & the [bootstrap-vault.sh](setup/bootstrap-vault.sh) for more details on both of these.
