# Pod Security Admissions (PSA)

In the previous section, we learned about Admission Controllers and how tools like Kyverno can block insecure Pods. 

However, ensuring that Pods are secure is so important that Kubernetes actually includes a built-in feature specifically for this, called **Pod Security Admission (PSA)**.

---

## 1. Why do we need it?

When you create a Docker container, it runs as `root` by default. If a hacker breaks into a container that is running as root, and that container is privileged, they might be able to break *out* of the container and take over the actual worker node (the EC2 instance or VM)!

To prevent this, you should enforce rules like:
- "Containers cannot run as root."
- "Containers cannot access the host network."
- "Containers cannot run in privileged mode."

**Pod Security Admission (PSA)** is the Kubernetes feature that enforces these rules.

---

## 2. The Three Security Levels

Instead of making you write a dozen different rules, Kubernetes has grouped security rules into three standard "Levels" (also called Profiles):

1. **Privileged (Level 1):** 
   - Completely open. No restrictions at all.
   - Use this only for system-level tools (like networking plugins) that actually need full access to the server.
2. **Baseline (Level 2):** 
   - Prevents the worst vulnerabilities (like host network access), but still allows normal applications to run without requiring you to rewrite them.
   - Good for most legacy applications.
3. **Restricted (Level 3):** 
   - Highly secure. Enforces all best practices (forces containers to run as a non-root user, drops all extra permissions).
   - Use this for all modern applications.

---

## 3. How to Apply PSA

The best part about PSA is how incredibly simple it is to use. You do not need to write complex YAML rules or install extra software. 

You apply PSA simply by **adding a label to a Namespace**.

For example, if we want to enforce the **Restricted** profile on a namespace called `my-app`, we just label it like this:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-app
  labels:
    # This single label enforces the Restricted profile!
    pod-security.kubernetes.io/enforce: restricted
```

### What happens next?
If a developer tries to deploy a standard NGINX pod (which runs as root by default) into the `my-app` namespace, the Kubernetes API Server will instantly reject the deployment and reply with:

*Error: pods "nginx" is forbidden: violates PodSecurity "restricted": runAsNonRoot (container must not run as root).*

The developer will then have to go update their Docker image and YAML file to run as a secure, non-root user before Kubernetes will allow the deployment.

---

## 4. Modes: Enforce vs Audit

If you add the `enforce` label to a namespace that already has running applications, and those applications break the rules, they might fail to restart!

To safely test rules before enforcing them, you can use **audit** or **warn** modes:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-app
  labels:
    # Just prints a warning to the developer's screen, but allows the Pod to run
    pod-security.kubernetes.io/warn: restricted
    
    # Logs a security event for the admin to review, but allows the Pod to run
    pod-security.kubernetes.io/audit: restricted
```

This allows you to see which applications violate the rules *before* you turn on `enforce: restricted`.

---

**Next →** [Kustomize](./09-kustomize.md)
