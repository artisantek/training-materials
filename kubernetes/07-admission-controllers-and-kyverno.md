# Admission Controllers & Kyverno

When you run `kubectl apply -f pod.yaml`, how does Kubernetes decide if you are actually allowed to create that Pod? 

The request goes through the **API Server** in three steps:
1. **Authentication:** "Who are you?" (Are you a valid user?)
2. **Authorization (RBAC):** "Are you allowed to create Pods?"
3. **Admission Control:** "Is this specific Pod safe and valid?"

**Admission Controllers** are the final gatekeepers (the "bouncers") in Kubernetes. Even if you have full admin permissions (RBAC), an Admission Controller can reject your request if it violates a rule.

---

## 1. Types of Admission Controllers

There are two main things an Admission Controller can do:

### A. Mutating (Changing the request)
Before the Pod is saved to the database, a Mutating Admission Controller can automatically change it.
- *Example:* If you forget to add a specific label to your Pod, the controller can automatically inject the label for you.
- *Example:* "Sidecar injection" (like automatically adding a proxy container to every Pod you create).

### B. Validating (Blocking the request)
After mutation, the Validating Admission Controller checks the final request. If it breaks a rule, it blocks the request and throws an error back to the user.
- *Example:* "You are not allowed to run a container as the `root` user."
- *Example:* "You cannot use the `latest` image tag in the production namespace."

---

## 2. The Problem with Custom Rules

By default, Kubernetes has several built-in admission controllers. But what if your company wants to create custom rules? For example, "Every Pod must have a `department` label."

Historically, to create a custom rule, you had to write a complex web server in Go or Python, deploy it to the cluster, and configure Kubernetes to talk to it. This was extremely hard for regular administrators to manage.

---

## 3. Enter Kyverno

**Kyverno** is a policy engine designed specifically for Kubernetes. It solves the complexity problem above by allowing you to write Admission Controller rules using simple YAML files!

With Kyverno, you don't need to write any code. You just write a `ClusterPolicy` in YAML.

### Example: A Validating Policy (Blocking 'latest' tags)

Here is how easy it is to write a rule in Kyverno that blocks anyone from using the `latest` image tag:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: block-latest-tag
spec:
  validationFailureAction: Enforce
  rules:
  - name: require-image-tag
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "You cannot use the 'latest' tag. Please use a specific version."
      pattern:
        spec:
          containers:
          - image: "!*:latest"
```

If a user tries to apply a Pod with `image: nginx:latest`, Kyverno will act as the Validating Admission Controller, block the request, and print the message: *"You cannot use the 'latest' tag. Please use a specific version."*

### Why Kyverno is Popular
1. **No Code Required:** Everything is standard Kubernetes YAML.
2. **Generates Resources:** It can automatically generate default ConfigMaps or Secrets when a new Namespace is created.
3. **Mutates Resources:** It can easily add default labels or annotations to resources.

---

**Next →** [Pod Security Admissions (PSA)](./08-pod-security-admissions.md)
