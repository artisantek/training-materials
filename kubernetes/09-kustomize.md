# Kustomize

When you are deploying an application, you usually need to run it in multiple environments: `development`, `staging`, and `production`. 

The YAML files for these environments are usually 90% identical. The only differences might be:
- In `development`, you want 1 Pod replica. In `production`, you want 5.
- In `development`, you connect to a test database. In `production`, you connect to the real database.

How do you manage these files without copy-pasting the same `deployment.yaml` three different times?

**Helm** solves this using complex Go templates. **Kustomize** solves this using a much simpler concept: **Overlays**.

---

## 1. What is Kustomize?

Kustomize is a configuration management tool built directly into `kubectl`. It allows you to define a "Base" set of YAML files, and then define "Overlays" that patch or modify that base for different environments.

There is no confusing template syntax (no `{{ .Values.something }}`). Everything in Kustomize is just plain YAML.

---

## 2. Base and Overlays

Here is how a Kustomize project is structured:

```text
my-app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        ├── kustomization.yaml
        └── replica-patch.yaml
```

### The `base` Folder
The `base` folder contains your standard, vanilla Kubernetes YAML files that apply to *all* environments. It also contains a special `kustomization.yaml` file that tells Kustomize which files to include.

**base/kustomization.yaml**:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

### The `overlays` Folder
The `overlays` folders (`dev`, `prod`) contain configurations that *modify* the base.

For example, in Production, we want to increase the replicas and add a `prod` label to everything. 

**overlays/prod/kustomization.yaml**:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 1. Point to the base files
resources:
  - ../../base

# 2. Add a common label to EVERY resource (Deployment, Service, Pods)
commonLabels:
  env: production

# 3. Apply a patch file to modify the base deployment
patchesStrategicMerge:
  - replica-patch.yaml
```

**overlays/prod/replica-patch.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  # This will override the replicas in the base deployment!
  replicas: 5
```

---

## 3. How to Apply Kustomize

Because Kustomize is built directly into Kubernetes, you don't need to install any extra tools to use it! 

To view the final, combined YAML for the production environment without actually deploying it (like a dry-run):
```bash
kubectl kustomize overlays/prod
```

To apply it to the cluster:
```bash
# Notice we use -k (for kustomize) instead of -f (for file)
kubectl apply -k overlays/prod
```

### Why use Kustomize over Helm?
1. **No templates to learn:** If you know Kubernetes YAML, you already know how to write Kustomize.
2. **Built-in:** You just use `kubectl apply -k`. You don't need to install the Helm CLI.
3. **Easier debugging:** The files are plain text, making it much easier to read than a Helm template filled with complex logic loops.

---

**Next →** [CSI Drivers](./10-csi-drivers.md)
