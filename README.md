# homelab-helm-charts

Small Helm charts I maintain myself for my homelab, for cases where the available
community chart is unmaintained or otherwise doesn't fit. Consumed directly from
ArgoCD via a git `path:` source — not published as a packaged Helm repo.

## Charts

### `charts/homepage`

A thin wrapper around [gethomepage/homepage](https://github.com/gethomepage/homepage),
built directly on the actively maintained
[bjw-s-labs common library](https://github.com/bjw-s-labs/helm-charts). Replaces
[jameswynn/helm-charts](https://github.com/jameswynn/helm-charts)' `homepage` chart,
which pins `common` to a version whose Route template hardcodes a Gateway API version
(`v1alpha2`) that current Gateway API installs (Cilium, etc.) don't serve for
`HTTPRoute`.

The RBAC (`ClusterRole`/`ClusterRoleBinding`) and six-file `ConfigMap` template logic in
`templates/_configmap.yaml` is adapted from `jameswynn/helm-charts` (Apache-2.0) — see
the attribution comment in that file. Everything else (`templates/common.yaml`) remaps
this chart's flat `image:`/`env:`/`config:` values onto `common`'s current nested
schema, the same way the original chart did against the old one.

Supported values (see `charts/homepage/values.yaml` for the full defaults):

| Key | Description |
|-----|-------------|
| `image.repository`, `image.tag` | Homepage image. |
| `env` | Standard container env list. |
| `enableRbac` | Grants the pod a ClusterRole for the Kubernetes widget (cluster/node stats, HTTPRoute/Ingress discovery). |
| `extraClusterRoles` | Extra PolicyRules appended to the generated ClusterRole. |
| `config.<bookmarks\|docker\|kubernetes\|services\|settings\|widgets>` (or `<key>String`) | Homepage's six config files. Use the `String` variant when key order matters (e.g. `settings.layout`) — `toYaml` sorts map keys alphabetically. |
| `config.useExistingConfigMap` | Skip generating the ConfigMap and mount an existing one instead. |
| `route`, `service`, `ingress`, and any other `common` key | Passed straight through to the bjw-s common library. |

Bumping `common` (`charts/homepage/Chart.yaml`) is tracked by Renovate — see
`renovate.json`.

## Consuming a chart from ArgoCD

```yaml
sources:
  - repoURL: https://github.com/BryanR77/homelab-helm-charts.git
    path: charts/homepage
    targetRevision: HEAD
    helm:
      valueFiles:
        - $values/homepage/values.yaml
  - repoURL: https://github.com/BryanR77/<app-values-repo>.git
    targetRevision: HEAD
    ref: values
```
