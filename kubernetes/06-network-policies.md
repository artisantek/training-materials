# Network Policies

When you deploy multiple applications into a Kubernetes cluster, a very important security feature to understand is how they communicate with each other.

By default, Kubernetes networking is completely open: **Any Pod can talk to any other Pod in the cluster, even across different namespaces.**

Imagine you have two apps: a `frontend` web server and a `backend` database. By default, if a hacker manages to compromise your frontend server, they can freely scan your network and connect directly to your backend database because there are no firewalls blocking them!

This is where **Network Policies** come in.

---

## 1. What is a Network Policy?

A **Network Policy** is exactly like a firewall, but for your Pods. It allows you to specify rules about what traffic is allowed to come IN (Ingress) to a Pod, and what traffic is allowed to go OUT (Egress) of a Pod.

When you apply a Network Policy to a Pod, that Pod becomes "isolated". It will reject all traffic *except* for the traffic you explicitly allow.

---

## 2. A Simple Example

Let's look at a very common scenario. We have a database Pod (labeled `app: database`). We want to protect it so that **only** the API Pod (labeled `app: api`) is allowed to connect to it. We want to block all other traffic.

Here is what the Network Policy would look like:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: protect-database
spec:
  # 1. Select the Pod we want to protect (the Database)
  podSelector:
    matchLabels:
      app: database
      
  # 2. Define the Ingress (Incoming traffic) rules
  ingress:
  - from:
    # 3. Allow traffic ONLY from Pods that have the 'app: api' label
    - podSelector:
        matchLabels:
          app: api
    # 4. Only allow connection on the database port (e.g., 3306)
    ports:
    - protocol: TCP
      port: 3306
```

### How to read this:
1. `podSelector`: This policy attaches to any Pod that has the label `app: database`.
2. `ingress.from`: It looks at incoming traffic. It checks if the incoming traffic is coming from a Pod labeled `app: api`.
3. `ports`: If the traffic is from the API Pod, it checks if it's trying to talk on port 3306. If yes, it allows it! If any other Pod (like the frontend) tries to connect, the connection is instantly blocked.

---

## 3. Important Note on Network Plugins (CNI)

Kubernetes itself **does not enforce** Network Policies. It just saves your rules to the database.

To actually enforce the firewall rules, your cluster must be running a networking plugin (called a CNI) that supports Network Policies.
- Examples of CNIs that **do** support them: Calico, Cilium, Weave Net.
- If your cluster does not use a supported CNI, you can create all the Network Policies you want, but Kubernetes will silently ignore them and leave your network open!

---

**Next →** [Admission Controllers & Kyverno](./07-admission-controllers-and-kyverno.md)
