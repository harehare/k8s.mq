<h1 align="center">k8s.mq</h1>

A [Kubernetes](https://kubernetes.io) manifest utility module for [mq](https://github.com/harehare/mq).

## Features

- Parse Kubernetes manifest YAML files
- Extract metadata: name, namespace, labels, annotations
- Access containers and initContainers (supports Pod, Deployment, DaemonSet, StatefulSet, Job, CronJob)
- Inspect images, environment variables, ports, and resource requests/limits
- Check manifest kind with convenience predicates
- Render manifest summaries as Markdown tables

## Installation

Copy `k8s.mq` to your mq module directory, or reference it with `-L`.

```sh
cp k8s.mq ~/.local/mq/config/
```

### HTTP Import (no local installation needed)

HTTP imports are disabled by default; pass `--allow-http-import` to import directly from GitHub without any local setup:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/k8s.mq" | k8s::k8s_parse(.) | k8s::k8s_images(.)' deployment.yaml
```

Pin to a specific release with `@vX.Y.Z`:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/k8s.mq@v0.1.0" | k8s::k8s_parse(.) | k8s::k8s_images(.)' deployment.yaml
```

## Usage

```sh
mq -L /path/to/modules -I raw \
  'import "k8s" | k8s::k8s_parse(.) | k8s::k8s_images(.)' deployment.yaml
```

## API

### Parsing

#### `k8s_parse(input)`

Parses a Kubernetes manifest YAML string and returns the parsed structure.

### Metadata

| Function | Description |
|---|---|
| `k8s_kind(manifest)` | Kind string (e.g. `"Deployment"`) |
| `k8s_api_version(manifest)` | `apiVersion` string |
| `k8s_name(manifest)` | Name from `metadata` |
| `k8s_namespace(manifest)` | Namespace from `metadata`, or None |
| `k8s_labels(manifest)` | Labels dict from `metadata` |
| `k8s_annotations(manifest)` | Annotations dict from `metadata` |
| `k8s_spec(manifest)` | Raw `spec` object |

### Containers

| Function | Description |
|---|---|
| `k8s_containers(manifest)` | Containers array (Pod, Deployment, DaemonSet, StatefulSet, Job, CronJob) |
| `k8s_init_containers(manifest)` | initContainers array |
| `k8s_images(manifest)` | All image strings (containers + initContainers) |
| `k8s_env(container)` | Env array from a container dict |
| `k8s_ports(manifest)` | All container ports |
| `k8s_resources(container)` | Resource requests/limits dict |

### Spec helpers

| Function | Description |
|---|---|
| `k8s_replicas(manifest)` | Replica count, or None |
| `k8s_selector(manifest)` | Selector dict from spec |

### Predicates

| Function | Description |
|---|---|
| `k8s_is_deployment(manifest)` | True if kind is `Deployment` |
| `k8s_is_service(manifest)` | True if kind is `Service` |
| `k8s_is_configmap(manifest)` | True if kind is `ConfigMap` |
| `k8s_is_pod(manifest)` | True if kind is `Pod` |

### Rendering

| Function | Description |
|---|---|
| `k8s_to_markdown_table(manifest)` | Markdown table summarizing the manifest |

## Example

Given `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
```

```sh
# Get all container images
mq -L . -I raw 'import "k8s" | k8s::k8s_parse(.) | k8s::k8s_images(.)' deployment.yaml
# => ["nginx:1.21"]

# Check if it is a Deployment
mq -L . -I raw 'import "k8s" | k8s::k8s_parse(.) | k8s::k8s_is_deployment(.)' deployment.yaml
# => true

# Render a Markdown summary
mq -L . -I raw 'import "k8s" | k8s::k8s_parse(.) | k8s::k8s_to_markdown_table(.)' deployment.yaml
```

## License

MIT
