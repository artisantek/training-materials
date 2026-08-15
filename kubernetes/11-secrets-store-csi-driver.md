# Secrets Store CSI Driver (AWS Secrets Manager & EKS)

When deploying applications to Kubernetes, managing sensitive data like database passwords, API tokens, and encryption keys securely is critical.

Traditionally, teams created standard Kubernetes Secrets. However, managing secrets across multiple tools creates security risks, manual sync overhead, and rotation headaches. 

The **Secrets Store CSI Driver** solves this by letting your Pods fetch secrets directly from **AWS Secrets Manager** as mounted files, using IAM permissions (IRSA) without needing any AWS credentials stored in your cluster.

---

## 1. How It Works

Instead of storing secrets inside Kubernetes `etcd`, the Secrets Store CSI Driver retrieves secrets on-demand from AWS Secrets Manager when a Pod starts up and mounts them as virtual files into the container.

```
┌─────────────────────────┐
│   AWS Secrets Manager   │  (Centralized Secret Store)
└────────────┬────────────┘
             │ 
             │ IRSA Authentication (OIDC)
             ▼
┌─────────────────────────┐
│ Secrets Store CSI Driver│  (DaemonSet)
│    + AWS Provider       │
└────────────┬────────────┘
             │ 
             │ Mounts as read-only files
             ▼
┌─────────────────────────┐
│       Pod Volume        │  (/mnt/secrets-store/DB_PASSWORD)
└────────────┬────────────┘
             │ 
             │ Read directly or export to environment
             ▼
┌─────────────────────────┐
│  Application Container  │  (App consumes credentials)
└─────────────────────────┘
```

### The 4 Core Building Blocks

1. **Secrets Store CSI Driver:** The standard Kubernetes plugin that manages mounting secret volumes to Pods.
2. **AWS Provider Plugin:** The driver extension that communicates with the AWS Secrets Manager API.
3. **IRSA (IAM Roles for Service Accounts):** Secure, passwordless authentication letting Pods assume an AWS IAM Role using standard OIDC tokens.
4. **SecretProviderClass:** A Kubernetes custom resource (CRD) where you define *which* AWS secret to fetch and *how* to expose its keys inside the Pod.

---

## 2. One-Time Cluster Setup

Before any application can use AWS secrets, set up the CSI driver and AWS provider on your EKS cluster.

### Step 1: Connect to your EKS Cluster
```bash
aws eks update-kubeconfig --region ap-south-1 --name movie-eks
```

### Step 2: Enable IRSA (IAM OIDC Provider)
Enable IAM Roles for Service Accounts on the cluster if not already enabled:
```bash
eksctl utils associate-iam-oidc-provider \
  --region ap-south-1 \
  --cluster movie-eks \
  --approve
```

### Step 3: Install the Secrets Store CSI Driver (via Helm)
```bash
# Add the official Helm repository
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm repo update

# Install the CSI Driver in the kube-system namespace
helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver \
  --namespace kube-system \
  --set syncSecret.enabled=false \
  --set enableSecretRotation=true
```

### Step 4: Install the AWS Provider Plugin
```bash
kubectl apply -f https://raw.githubusercontent.com/aws/secrets-store-csi-driver-provider-aws/main/deployment/aws-provider-installer.yaml
```

### Step 5: Patch the CSI Driver for Token Requests
Some Kubernetes / Helm versions require explicit token request permissions to exchange service account tokens with AWS STS:
```bash
kubectl patch csidriver secrets-store.csi.k8s.io --type='json' -p='[
  {"op":"add","path":"/spec/tokenRequests","value":[{"audience":"sts.amazonaws.com","expirationSeconds":3600}]}
]'
```

---

## 3. Hands-On: Mounting an AWS Secret in a Pod

Let's walk through a complete example: creating a secret in AWS Secrets Manager and accessing it inside a Kubernetes Pod.

### Step 1: Create a Secret in AWS Secrets Manager

Create a secret holding our database credentials:
```bash
aws secretsmanager create-secret \
  --name movie-eks/db-credentials \
  --region ap-south-1 \
  --secret-string '{"username":"studentuser","password":"SuperSecret123"}'
```

### Step 2: Create an IAM Policy Scoped to the Secret

Create a policy that allows reading only this specific secret:

```bash
cat <<EOF > secrets-policy.json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "secretsmanager:GetSecretValue",
      "secretsmanager:DescribeSecret"
    ],
    "Resource": ["arn:aws:secretsmanager:ap-south-1:070797855002:secret:movie-eks/*"]
  }]
}
EOF

aws iam create-policy \
  --policy-name movie-eks-secrets-csi-policy \
  --policy-document file://secrets-policy.json
```

### Step 3: Create the IAM Role for IRSA

Fetch the cluster's OIDC ID and create a trust policy binding the IAM Role to the Kubernetes ServiceAccount:

```bash
# 1. Get cluster OIDC provider URL ID
OIDC_ID=$(aws eks describe-cluster --name movie-eks --region ap-south-1 \
  --query "cluster.identity.oidc.issuer" --output text | sed 's|https://||')

# 2. Define the trust relationship policy
cat <<EOF > trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Federated": "arn:aws:iam::070797855002:oidc-provider/${OIDC_ID}"
    },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "${OIDC_ID}:sub": "system:serviceaccount:secrets-test:movie-app-sa",
        "${OIDC_ID}:aud": "sts.amazonaws.com"
      }
    }
  }]
}
EOF

# 3. Create the IAM Role and attach the secret policy
aws iam create-role \
  --role-name movie-app-sa-role \
  --assume-role-policy-document file://trust-policy.json

aws iam attach-role-policy \
  --role-name movie-app-sa-role \
  --policy-arn arn:aws:iam::070797855002:policy/movie-eks-secrets-csi-policy
```

### Step 4: Create Namespace and ServiceAccount

Create the namespace and annotate the ServiceAccount with the IAM Role ARN:

```bash
# Create namespace
kubectl create namespace secrets-test

# Create ServiceAccount
kubectl create serviceaccount movie-app-sa -n secrets-test

# Annotate ServiceAccount with the IAM Role ARN
kubectl annotate serviceaccount movie-app-sa -n secrets-test \
  eks.amazonaws.com/role-arn=arn:aws:iam::070797855002:role/movie-app-sa-role
```

### Step 5: Define the SecretProviderClass

The `SecretProviderClass` tells the CSI driver which secret to fetch from AWS and maps the JSON keys to file names (aliases).

Create `secretproviderclass.yaml`:
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: movie-app-secrets
  namespace: secrets-test
spec:
  provider: aws
  parameters:
    objects: |
      - objectName: "movie-eks/db-credentials"
        objectType: "secretsmanager"
        jmesPath:
          - path: "username"
            objectAlias: "DB_USERNAME"
          - path: "password"
            objectAlias: "DB_PASSWORD"
```

Apply it:
```bash
kubectl apply -f secretproviderclass.yaml
```

### Step 6: Deploy the Test Pod

Create `simple-secrets-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secrets-test-pod
  namespace: secrets-test
spec:
  serviceAccountName: movie-app-sa
  volumes:
    # 1. Define the CSI Volume pointing to the SecretProviderClass
    - name: secrets-store
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: movie-app-secrets
  containers:
    - name: busybox
      image: busybox:1.36
      command:
        - sh
        - -c
        - |
          # Read all files in mount directory and export as environment variables
          for f in /mnt/secrets-store/*; do
            n=$(basename "$f" | tr -c 'A-Za-z0-9_' '_')
            export "$n=$(cat $f)"
          done
          echo "Secrets loaded successfully into container environment."
          sleep infinity
      volumeMounts:
        # 2. Mount the volume into the container
        - name: secrets-store
          mountPath: /mnt/secrets-store
          readOnly: true
```

Apply and monitor the Pod:
```bash
kubectl apply -f simple-secrets-pod.yaml
kubectl get pod secrets-test-pod -n secrets-test -w
```

---

## 4. Verifying the Secrets

Once the Pod is running (`Running` status), verify that the secrets are accessible inside:

### 1. View Mounted Secret Files
```bash
# List the secret files
kubectl exec -it secrets-test-pod -n secrets-test -- ls -la /mnt/secrets-store

# Read the username & password directly from the mounted files
kubectl exec -it secrets-test-pod -n secrets-test -- cat /mnt/secrets-store/DB_USERNAME
kubectl exec -it secrets-test-pod -n secrets-test -- cat /mnt/secrets-store/DB_PASSWORD
```

### 2. Verify Environment Variables
```bash
kubectl exec -it secrets-test-pod -n secrets-test -- sh -c '
for f in /mnt/secrets-store/*; do
  n=$(basename "$f" | tr -c "A-Za-z0-9_" "_")
  export "$n=$(cat $f)"
done
env | grep DB_
'
```

---

## 5. Important Concept: Process Isolation & `kubectl exec`

> 💡 **Why doesn't `kubectl exec` show exported environment variables automatically?**
>
> When you run `kubectl exec`, Kubernetes opens a **brand-new, independent shell process** inside the container. 
> 
> Variables exported in the container's startup script (`PID 1`) only exist within that process and its children. A new `kubectl exec` session does not inherit them.

### Production Patterns for Consuming Secrets:
1. **Direct File Reading (Best Practice):** Configure your application framework (e.g., Spring Boot, Node.js, Django) to read credentials directly from `/mnt/secrets-store/<secret_name>`.
2. **Wrapper Entrypoint Script:** Make your container's `ENTRYPOINT` script load/export the files before launching the main app process (`exec "$@"`).
3. **Init Container / Shared Volume:** Mount the CSI volume in an init container, write parsed environment files to a shared volume, and source them on application boot.

---

## 6. Common Errors & Troubleshooting

| Error | Root Cause | Solution |
|---|---|---|
| `CSI token error: serviceAccount.tokens not provided` | CSI Driver missing `tokenRequests` capability | Patch the CSIDriver resource (see Section 2, Step 5). |
| `AccessDenied ... cloudformation:ListStacks` | `eksctl create iamserviceaccount` lacks CloudFormation permissions | Create the IAM Role and Trust Policy manually using the AWS CLI. |
| `export: bad variable name` | Secret keys contain hyphens (`-`) or slashes (`/`), invalid in POSIX shell | Sanitize variable names using `tr -c 'A-Za-z0-9_' '_'`. |
| `/bin/sh: source: not found` | Minimal images (Alpine / Busybox) run `sh`, where `source` is not built-in | Use `.` (dot) instead of `source` in POSIX `sh`. |
| `container not found` during `kubectl exec` | Container crashed or exited immediately on start | Inspect logs via `kubectl logs <pod> --previous`, or ensure a long-running process (like `sleep infinity`) keeps the container alive. |

---

## Summary of Key Takeaways

- **No Hardcoded Credentials:** IRSA enables secure IAM authentication directly using OIDC without saving AWS access keys in Kubernetes.
- **On-Demand Mounting:** Secrets are fetched from AWS Secrets Manager only when Pods mount the CSI volume.
- **Granular Least-Privilege:** IAM policies can be restricted to specific secret ARN paths (`arn:aws:secretsmanager:...:secret:movie-eks/*`).
- **Rotation Support:** With `enableSecretRotation=true`, the CSI driver automatically polls AWS Secrets Manager for updated secret values.

---

**Next →** [Return to Main Menu](../README.md)
