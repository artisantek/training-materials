# Advanced Helm Templating

While basic Helm usage involves simple variable replacement (like `{{ .Values.image }}`), real-world Helm charts require dynamic logic. This guide covers the advanced features of Helm that are critical for enterprise deployments: control structures, loops, scope management, and advanced template functions.

---

## 1. Minimal Introduction to Helm

Helm is the package manager for Kubernetes. A **Helm Chart** is a collection of templates that generate Kubernetes YAML manifests. Helm uses the **Go Templating Engine**, which means you can write logic (if/else, loops) directly inside your YAML files. The values for these templates are usually provided by a `values.yaml` file.

---

## 2. Control Structures (`if` / `else`)

Helm allows you to conditionally create entire Kubernetes resources or just pieces of YAML.

### Basic Syntax
Imagine you only want to create an Ingress if the user asks for it in `values.yaml`:

```yaml
{{- if .Values.createIngress }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
{{- end }}
```

### Whitespace Control (`-`)
Notice the hyphen (`-`) in `{{- if ... }}` and `{{- end }}`. 
If you omit the hyphen, Helm will render an empty blank line in your YAML file where the `if` statement was evaluated, which can cause YAML formatting errors.
- `{{- ` removes all whitespace **before** the block.
- ` -}}` removes all whitespace **after** the block.

---

## 3. Loops (`range`) and Scope Shifting

The `range` function is used to loop over lists or maps (dictionaries) in your `values.yaml`.

### Iterating over a List
If `values.yaml` has a list of environments:
```yaml
environments:
  - dev
  - qa
  - prod
```

You can loop through them in your YAML. Inside the loop, the dot (`.`) represents the current item:
```yaml
envList:
{{- range .Values.environments }}
  - {{ . }}
{{- end }}
```

### Iterating over a Map (Key/Value)
If `values.yaml` has a dictionary of custom labels:
```yaml
customLabels:
  team: backend
  tier: api
```

You can loop through the keys and values like this:
```yaml
metadata:
  labels:
  {{- range $key, $value := .Values.customLabels }}
    {{ $key }}: {{ $value }}
  {{- end }}
```

### ⚠️ The Scope Problem (`$`)
This is the most common mistake in Helm! 

Inside a `range` loop, the meaning of the dot (`.`) **changes**. It no longer points to the root of your chart; it points to the current item in the loop. 

If you try to use `{{ .Release.Name }}` inside a loop, **Helm will crash** because the current item doesn't have a `.Release` property. To fix this, use the `$` symbol, which always points to the absolute root:

```yaml
{{- range .Values.environments }}
  # Use . for the list item, and $ for the root!
  - name: {{ . }}-{{ $.Release.Name }}
{{- end }}
```

---

## 4. Named Templates (`define`)

If you are typing the same labels in your Deployment, Service, and Ingress, you can create a reusable "function" in a `_helpers.tpl` file.

**Step 1: Define the template (in `_helpers.tpl`)**
```gotemplate
{{- define "my-standard-labels" -}}
app: my-cool-app
managed-by: helm
{{- end -}}
```

**Step 2: Use it (in `deployment.yaml`)**
```yaml
metadata:
  labels:
    {{- include "my-standard-labels" . | nindent 4 }}
```

> 💡 **`include` vs `template`:** You will see both online, but you should always use `include`. Why? Because `template` is an *action*, not a function, meaning it doesn't return a value that can be piped. `include` is a function that returns the rendered string, allowing you to pipe the output directly to `nindent 4` (to automatically fix the YAML spacing).

### Pro-Tip: `toYaml` and `nindent`
This exact same piping pattern is heavily used with the `toYaml` function. If a user passes an entire block of config in `values.yaml` (like CPU/Memory limits), you can dump the whole block into your manifest and fix its indentation in one line:
```yaml
resources:
  {{- toYaml .Values.resources | nindent 2 }}
```

---

## 5. Guarding Values (`default` & `required`)

When building reusable charts, you cannot guarantee users will provide every value. Helm provides two critical functions for this:

1. **`default`**: Provides a fallback value if one isn't set.
```yaml
# If .Values.replicaCount is missing, it defaults to 1
replicas: {{ .Values.replicaCount | default 1 }}
```

2. **`required`**: Fails the deployment and throws a custom error if a mandatory value is missing. This is heavily paired with advanced functions to ensure users pass the right data.
```yaml
# The chart will crash and print this error if dbPassword is missing
password: {{ required "A valid .Values.dbPassword is required!" .Values.dbPassword }}
```

---

## 6. The Magic of `tpl`

The `tpl` function is a very advanced trick. Normally, `values.yaml` is just dumb, static text. But `tpl` allows you to write **Helm template code inside `values.yaml`** and have Helm compile it on the fly!

**In `values.yaml`:**
```yaml
# We wrote template code in a standard string!
welcomeMessage: "Hello, this is release {{ .Release.Name }}!"
```

If you use `{{ .Values.welcomeMessage }}`, Helm will just print the exact text without replacing the variables. But if you pass it through `tpl`, Helm evaluates it:

**In `configmap.yaml`:**
```yaml
data:
  # This tells Helm: "Treat this string as template code and compile it!"
  message: {{ tpl .Values.welcomeMessage . | quote }}
```

**Output:**
```yaml
data:
  message: "Hello, this is release production-app!"
```
This is heavily used in enterprise charts to let users inject dynamic values into environments and configmaps!

---

## 7. Changing Scope with `with`

If you have deeply nested values in your `values.yaml`, typing the full path over and over is annoying:
```yaml
# values.yaml
database:
  credentials:
    username: admin
    password: secretpassword
```

Instead of typing `{{ .Values.database.credentials.username }}`, you can use the `with` block to temporarily shift the scope (`.`) to that block:

```yaml
{{- with .Values.database.credentials }}
username: {{ .username }}
password: {{ .password }}
{{- end }}
```
> ⚠️ **Warning:** Just like `range`, the `with` block changes your scope! Inside the `with` block, you cannot access `.Release` unless you use `$.Release`.

---

## 8. Helm Hooks (Running DB Migrations)

Sometimes you need to run a task *before* your pods start, or *after* they are deleted. A classic example is running a Database Migration Job before upgrading your web app.

You can do this by adding a special annotation to a Kubernetes `Job`:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    # Tell Helm to run this BEFORE the rest of the chart is upgraded/installed
    "helm.sh/hook": pre-install,pre-upgrade
    # Controls execution order (lower numbers run first) if you have multiple hooks!
    "helm.sh/hook-weight": "-5"
    # Tell Helm to delete the Job after it succeeds so it doesn't clutter the cluster
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: my-db-migrator:v1
```

---

## 9. Querying the Live Cluster (`lookup`)

By default, Helm just renders text templates and sends them to Kubernetes. But what if your template needs to know if a resource *already exists* in the cluster before deciding what to do? 

The `lookup` function allows Helm to ask the Kubernetes API for live information during the installation!

```yaml
# Ask Kubernetes if a secret named "my-secret" exists in the "default" namespace
{{- $mySecret := lookup "v1" "Secret" "default" "my-secret" }}

# If the secret DOES NOT exist, create it
{{- if not $mySecret }}
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  password: "super-secret-default-password"
{{- end }}
```
> ⚠️ **CI/CD Gotcha:** When you run `helm template` or `helm dry-run`, there is no live cluster to query. `lookup` won't error out—it will just silently return an empty dictionary. This means the `{{- if not $mySecret }}` block above will **always** render during a dry-run/template in your CI pipeline, even if the secret already exists in production!
