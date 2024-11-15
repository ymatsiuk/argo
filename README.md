## Bootstrap

```
kind create cluster
helm install argocd argo/argo-cd -f values.yaml
kubectl apply -f projects.yaml
kubectl apply -f rootApp.yaml
```

## How It Works

This repository utilizes the `ApplicationSet` kind in ArgoCD to manage applications. It supports both upstream Helm charts and local charts, allowing for flexibility in deployment.

The full `ApplicationSet` configuration can be found under [`apps/apps.yaml`](apps/apps.yaml).

### Helm Charts

- **Values Precedence**: Values for Helm charts are defined in two primary locations:
  1. `applications/<project>/<app>/values.yaml`
  2. `applications/<project>/<app>/<env>/values.yaml`

  These files are used to override default values, ensuring environment-specific configurations.

### Loose Resources Deployment

- **Loose Resources**: Resources that are not part of a Helm chart but still need to be deployed are included directly from the repository. ArgoCD will recursively traverse the directory structure starting from `../<env>/`, which means you can organize your resources in either directories or just dump them all into one file alongside your `values.yaml`.

  Note: Deployment of loose resources **without** `chart.yaml` is not currently supported. An `Application` will only be created if `chart.yaml` is present (check the git generator inside `ApplicationSet`).

### Structure

- **Directory Structure**:
  ```
  applications/<project>/<app>/<env>/chart.yaml
  ```

  This structure allows for easy management of multiple applications. Each project contains multiple applications that can be deployed into multiple environments, supporting overrides per environment. The existence of `../<env>/..` denotes that the application must be deployed to an `<env>`.

### `chart.yaml`

The `chart.yaml` file specifies the details of the Helm chart to be deployed. It includes the following fields:

- `repoURL`: The URL of the Helm chart repository.
- `name`: The name of the Helm chart. This is only required for upstream Helm and not used for local charts.
- `version`: The version of the Helm chart. This is only required for upstream Helm and defaults to `HEAD` for local charts.
- `namespace`: The Kubernetes namespace where the chart will be deployed. Defaults to the application name derived from the path.
- `releasename`: The release name for the Helm chart. Defaults to the application name derived from the path.
- `serversideapply`: An application [sync option](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/#server-side-apply) that doesn't rely on annotations. Defaults to `false` if not specified.

#### Example Upstream Chart

This example deploys the `prometheus-blackbox-exporter` Helm chart from the Prometheus community repository:

```yaml
chart:
  repoURL: https://prometheus-community.github.io/helm-charts
  name: prometheus-blackbox-exporter
  version: "9.*.*"
  namespace: monitoring
  releasename: blackbox-exporter
```

#### Example Local Chart

This example deploys a local chart from the `charts/namespaces` directory:

```yaml
chart:
  repoURL: https://github.com/ymatsiuk/argo.git
  path: charts/namespaces
  namespace: default
  serversideapply: true
```

## Contributing

### Adding Your Application

1. **Create Your Chart Directory**:
   - Navigate to `applications/<project>/<app>/<env>/`.
   - Add your `chart.yaml` file in this directory.

     Note: This step is mandatory; if `chart.yaml` doesn't exist, the directory is skipped.

2. **Define Values**:
   - Add a `values.yaml` file in `applications/<project>/<app>/`.
   - Add another `values.yaml` in `applications/<project>/<app>/<env>/` for environment-specific configurations.

     Note: These files are optional; upstream defaults should be used if both are missing.

### Example Application Structure

```
applications/myproject/myapp/dev/chart.yaml
applications/myproject/myapp/values.yaml
applications/myproject/myapp/dev/values.yaml
applications/myproject/myapp/dev/rolebindings.yaml
```
