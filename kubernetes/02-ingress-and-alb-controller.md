# Ingress and AWS ALB Controller

In Kubernetes, a Service (like NodePort or LoadBalancer) exposes your application, but it operates at Layer 4 (TCP/UDP). What if you need Layer 7 (HTTP/HTTPS) routing, path-based routing, SSL termination, or virtual host routing? 

This is where **Ingress** comes in.

---

## What is Ingress?

An **Ingress** is an API object that manages external access to the services in a cluster, typically HTTP. Ingress can provide load balancing, SSL termination, and name-based virtual hosting.

However, creating an Ingress resource alone does nothing. You must have an **Ingress Controller** running in your cluster to fulfill the Ingress rules.

### Ingress Classes

Because you can have multiple Ingress Controllers running in a single cluster, Kubernetes uses `IngressClass` to specify which controller should process a specific Ingress object.

Common Ingress Controllers:
- `nginx` (NGINX Ingress Controller)
- `alb` (AWS Load Balancer Controller)
- `haproxy` (HAProxy Ingress)
- `traefik` (Traefik Ingress)

> ⚠️ **Important Note on `ingress-nginx`:** The community-maintained `ingress-nginx` project is effectively **discontinued with respect to active feature development** and reaches End of Life (EOL) in March 2026. 
> 
> The shift away from it by enterprises is driven by three main factors:
> 1. **End of Support:** After March 2026, it will receive no further security patches or bug fixes.
> 2. **API Feature Freeze:** The Kubernetes Ingress API itself is feature-frozen, with the community shifting all focus to the newer **Gateway API**.
> 3. **Operational Overhead:** `ingress-nginx` requires running a fleet of NGINX pods *and* a separate Layer 4 Load Balancer. Cloud-native alternatives like the **AWS Load Balancer Controller** cut out the middleman by directly provisioning managed Layer 7 ALBs, heavily reducing operational complexity.

---

## AWS Load Balancer Controller (ALB Controller)

The AWS Load Balancer Controller manages AWS Elastic Load Balancers for a Kubernetes cluster. 

When you create a Kubernetes `Ingress` with the `alb` ingress class, the controller provisions an **Application Load Balancer (ALB)** in AWS.

### How it Works

1. **Ingress Creation:** You create an Ingress resource with `spec.ingressClassName: alb` (or the legacy `kubernetes.io/ingress.class: alb` annotation).
2. **ALB Provisioning:** The controller watches for this and creates an AWS ALB.
3. **Target Groups:** It creates AWS Target Groups and registers your Kubernetes Pod IPs (or Node IPs) as targets.
4. **Listeners & Rules:** It configures the ALB listeners and rules based on the paths defined in your Ingress manifest.

### Subnet Tagging Requirements

Before installing the controller, your AWS VPC subnets must be properly tagged so the controller can auto-discover them and provision the ALB:
- **Cluster Tag (All Subnets):** Must have the tag `kubernetes.io/cluster/<cluster-name> = shared` or `owned` (e.g., `kubernetes.io/cluster/eks-cluster = shared`).
- **Public Subnets** (for internet-facing ALBs): Must also have the tag `kubernetes.io/role/elb = 1`.
- **Private Subnets** (for internal ALBs): Must also have the tag `kubernetes.io/role/internal-elb = 1`.

### Installing the AWS Load Balancer Controller

To allow the controller to manage resources on behalf of your Kubernetes cluster, it needs AWS permissions. We achieve this by linking an IAM policy to a Kubernetes service account using IRSA (IAM Roles for Service Accounts).

#### 1. IAM Policy and Service Account

First, download the official IAM policy JSON file and create a policy in AWS:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

Next, create an IAM service account using `eksctl` (replace the AWS account ID in the ARN with your own):

```bash
eksctl create iamserviceaccount \
  --cluster=eks-cluster \
  --namespace=alb \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::<YOUR_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region=ap-south-1
```

#### 2. Install using Helm Chart

Add the EKS Helm chart repository:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```

Install the AWS Load Balancer Controller using Helm:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n alb --create-namespace \
  --set clusterName=eks-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set vpcId=<YOUR_VPC_ID> \
  --set region=ap-south-1
```

---

### Key Annotations for ALB Controller

The true power of the ALB controller lies in its annotations. You configure almost everything about the ALB directly from the Kubernetes YAML.

Here are the most critical annotations you need to know:

| Annotation | Description | Example |
|------------|-------------|---------|
| `alb.ingress.kubernetes.io/scheme` | Defines whether the ALB is public (`internet-facing`) or private (`internal`). | `internet-facing` |
| `alb.ingress.kubernetes.io/target-type` | How traffic is routed. `instance` routes to Node IPs (requires NodePort). `ip` routes directly to Pod IPs (recommended for EKS). | `ip` |
| `alb.ingress.kubernetes.io/listen-ports` | The ports the ALB should listen on. | `'[{"HTTP": 80}, {"HTTPS":443}]'` |
| `alb.ingress.kubernetes.io/certificate-arn` | ARN of the ACM Certificate to use for HTTPS. **Must be in the same region as your EKS cluster.** | `arn:aws:acm:ap-south-1:1234:certificate/...` |
| `alb.ingress.kubernetes.io/ssl-redirect` | Automatically redirects HTTP (port 80) traffic to HTTPS (port 443). | `'443'` |
| `alb.ingress.kubernetes.io/healthcheck-path` | The path the ALB uses to check if your Pods are healthy. Crucial to avoid 502 Bad Gateway errors. | `/health` |
| `alb.ingress.kubernetes.io/subnets` | Specific subnets where the ALB should be provisioned (can also use auto-discovery tags). | `subnet-0a1b2c3d, subnet-4e5f6g7h` |
| `alb.ingress.kubernetes.io/security-groups` | Attach an existing AWS Security Group to the ALB. | `sg-0123456789abcdef0` |

---

## Example: ALB Ingress Manifest

Here is a complete example of a production-ready Ingress object using the AWS Load Balancer Controller:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: my-app-namespace
  annotations:
    # Make the ALB public
    alb.ingress.kubernetes.io/scheme: internet-facing
    
    # Route traffic directly to Pod IPs (Requires AWS VPC CNI)
    alb.ingress.kubernetes.io/target-type: ip
    
    # Listen on HTTP and HTTPS
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    
    # Redirect HTTP to HTTPS
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    
    # Configure Health Checks to avoid 502 errors
    alb.ingress.kubernetes.io/healthcheck-path: '/'
    
    # Attach ACM Certificate (MUST match the cluster's region, e.g., ap-south-1)
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:123456789012:certificate/xyz-123
spec:
  # Use the modern ingressClassName instead of the legacy annotation
  ingressClassName: alb
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

### Why this is powerful
With this single file, you are not just configuring Kubernetes routing; you are dynamically provisioning and configuring AWS infrastructure (ALB, Target Groups, Listeners, Rules, SSL) via Infrastructure as Code (IaC) natively within Kubernetes.

---

**Next →** [Gateway API](./03-gateway-api.md)
