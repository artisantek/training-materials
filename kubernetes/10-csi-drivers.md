# CSI Drivers (Container Storage Interface)

When a Pod needs to save data permanently (like a Database storing its tables), it needs a **Volume** (a virtual hard drive). 

In the early days of Kubernetes, the code that allowed Kubernetes to talk to cloud providers (like AWS EBS, Google Persistent Disks, or Azure Disks) was baked directly into the Kubernetes source code. This was called **"in-tree"** storage.

This caused a huge problem: If AWS wanted to fix a bug in their EBS hard drives, they had to wait for the next official Kubernetes release to launch the fix!

---

## 1. What is a CSI Driver?

To solve this problem, Kubernetes created the **Container Storage Interface (CSI)**. 

CSI is a standard API that allows storage providers (AWS, Google, NetApp, VMware) to write their own driver plugins and install them onto a Kubernetes cluster **without modifying the core Kubernetes code.** This is known as **"out-of-tree"** storage.

When you install a CSI Driver on your cluster (for example, the `aws-ebs-csi-driver`), it acts as a translator between Kubernetes and the AWS Cloud.

---

## 2. How it works (The 3 Concepts)

To request a hard drive in Kubernetes using a CSI driver, you use three main resources:

### A. StorageClass (The Blueprint)
The StorageClass tells Kubernetes *which* CSI driver to use, and what type of drive to create. 

For example, on AWS, you might create a StorageClass for a fast SSD:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
# 1. This tells Kubernetes to talk to the AWS EBS CSI Driver
provisioner: ebs.csi.aws.com
parameters:
  # 2. Tell AWS to make it a fast SSD (gp3)
  type: gp3
```

### B. Persistent Volume Claim (PVC) - (The Request)
When a developer writes an application, they don't care if it's running on AWS, Google, or a local server. They just know they need 10 Gigabytes of fast storage.

They create a **PVC** to ask for it:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-database-storage
spec:
  # Point to the blueprint we created above!
  storageClassName: fast-ssd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### C. Persistent Volume (PV) - (The Actual Hard Drive)
When Kubernetes sees the PVC request, it looks at the `StorageClass`. 
1. It sees the provisioner is `ebs.csi.aws.com`.
2. It sends a message to the AWS CSI Driver.
3. The AWS CSI Driver talks to the real AWS Cloud and creates a real 10GB EBS volume.
4. Kubernetes automatically creates a **Persistent Volume (PV)** object to represent that real drive, and binds it to the user's PVC.

---

## 3. Using the Storage in a Pod

Now that the PVC has successfully requested the storage, you can mount it into your Pod!

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-database
spec:
  containers:
  - name: db
    image: postgres
    volumeMounts:
    - mountPath: "/var/lib/postgresql/data"
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      # Tell the Pod to use the PVC we just created!
      claimName: my-database-storage
```

---

## Summary of the Flow
1. Developer creates a **PVC** asking for 10GB of `fast-ssd` storage.
2. The **StorageClass** tells the **CSI Driver** to talk to the Cloud Provider.
3. The Cloud Provider creates a real hard drive.
4. Kubernetes creates a **PV** to represent the hard drive.
5. The **Pod** mounts the PVC and starts using it.

---

**Next →** [Secrets Store CSI Driver (AWS Secrets Manager)](./11-secrets-store-csi-driver.md)
