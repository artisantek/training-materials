# Kubernetes Gateway API

> 📺 **Video Tutorial:** You can follow along with the concepts and demonstrations in this [YouTube Tutorial on Gateway API](https://www.youtube.com/watch?v=xaZ87iSvMAI).

While the traditional `Ingress` API has been the standard for routing external traffic into a Kubernetes cluster for years, it has severe limitations that become obvious in enterprise environments. To solve these limitations, the Kubernetes community introduced the **Gateway API**.

---

## The Problem with Ingress

Before the Gateway API, Ingress was the primary way to expose services outside the cluster. However, it created several major challenges:

### 1. Lack of Multi-Tenancy
Imagine a cluster where one team manages the `wear` service (clothing) and another team manages the `watch` service. With Ingress, there was no native support for multi-tenancy or namespace isolation. Both teams had to coordinate and often work on the exact same Ingress resource, resulting in them stepping on each other's toes.

### 2. Annotation Soup for Advanced Routing
The `spec` section of a native Ingress resource only supports basic host-matching (`wear.myonlinestore.com`) and path-matching (`/wear`). It does not natively support:
- TCP/UDP Routing
- Traffic Splitting (Canary Deployments)
- Header Manipulation, Redirects, or Rate Limiting
- CORS or Session Affinity

To bypass this, administrators had to use **Annotations** (e.g., `nginx.ingress.kubernetes.io/rewrite-target`). Because Kubernetes does not understand annotations (it just passes them to the controller), these configurations were inconsistent. An annotation written for an NGINX controller would not work if the company migrated to Traefik, requiring massive rewrites.

---

## What is the Gateway API?

The Gateway API is the next generation of Ingress and Load Balancing in Kubernetes. It focuses on Layer 4 and Layer 7 routing and brings advanced features directly into the native Kubernetes specification.

### Core Concepts (The Three Personas)

The Gateway API splits routing into three separate resources, allowing different roles in an organization to manage only what they are responsible for:

1. **`GatewayClass` (Infrastructure Provider):** 
   - Defines *how* a Gateway is implemented by a controller (e.g., NGINX Gateway Fabric, AWS ALB, Traefik). 
   - Typically managed by cloud providers or platform architects.

2. **`Gateway` (Cluster Operator):**
   - Represents the actual traffic handling infrastructure (like a Load Balancer instance). 
   - Defines the ports, listeners, and TLS configurations. 
   - Managed by DevOps/SysAdmins.

3. **`HTTPRoute` / `TCPRoute` (Application Developer):**
   - Defines the rules for routing traffic to specific backend services. 
   - Attaches to a parent `Gateway`.
   - Managed directly by the developers building the application.

> 💡 **Why this is better:** Unlike Ingress, `HTTPRoute` moves features like **Canary Deployments** (traffic weights) and **CORS** out of annotations and directly into the API `spec` rules.

---

## Example: Deploying with Gateway API

Notice how the infrastructure definition is completely separated from the application routing.

### Step 1: The Platform Team deploys the Gateway
The DevOps team creates a single shared Gateway for the cluster. Notice the `allowedRoutes` property, which explicitly allows `HTTPRoutes` from *all* namespaces to attach to this Gateway, solving the multi-tenancy issue.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: shared-gateway
  namespace: infrastructure
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All
```

### Step 2: The Developers attach their Routes
Developers in different teams can now create `HTTPRoutes` that attach to that `shared-gateway`, routing traffic to their specific services without having to manage the load balancer infrastructure.

**Team A (Backend Team):**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: backend-route
  namespace: backend-team
spec:
  parentRefs:
  - name: shared-gateway
    namespace: infrastructure
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-service
      port: 8080
```

**Team B (Frontend Team):**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: frontend-route
  namespace: frontend-team
spec:
  parentRefs:
  - name: shared-gateway
    namespace: infrastructure
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80
```

By applying this, traffic hitting the central Gateway will be intelligently routed to either `api-service` in the `backend-team` namespace or `web-service` in the `frontend-team` namespace. The infrastructure is completely decoupled from the application routing!

---

**Next →** [Return to Main Menu](../README.md)
