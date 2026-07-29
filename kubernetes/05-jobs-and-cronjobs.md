# Jobs and CronJobs

So far, we have looked at **Deployments**, which are designed to keep your applications (like web servers) running constantly. If a Pod in a Deployment stops or crashes, Kubernetes immediately restarts it to ensure it is always running.

But what if you have a task that is **supposed** to finish and stop? 
For example:
- Running a database migration script.
- Processing a batch of images.
- Running a nightly backup.

You don't want Kubernetes to endlessly restart a backup script after it successfully finishes. This is where **Jobs** and **CronJobs** come in.

---

## 1. What is a Job?

A **Job** creates one or more Pods and ensures that a specified number of them successfully terminate (complete their task). 

When the container inside the Pod exits with a success code (`0`), the Job is considered complete. The Pod is not restarted; it just moves to a `Completed` state.

### Example: A Simple Job
Here is a Job that runs a simple math calculation and then exits:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-calculation
spec:
  template:
    spec:
      containers:
      - name: calculator
        image: busybox
        # This is the task the container will run
        command: ["sh", "-c", "echo 'Calculating...'; sleep 5; echo '2 + 2 = 4';"]
      # Notice the restartPolicy is NOT 'Always' like a Deployment
      restartPolicy: Never
```

If you apply this file (`kubectl apply -f job.yaml`), Kubernetes will run the Pod. After 5 seconds, it prints the answer and stops. If you run `kubectl get pods`, you will see the Pod's status as `Completed`.

---

## 2. What is a CronJob?

A **CronJob** is exactly like a Job, but it runs on a repeating time schedule. It acts just like the `cron` utility in Linux.

It is perfect for tasks like sending out weekly email reports or running a database backup every night at 2:00 AM.

### Example: A Nightly Backup CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  # This uses standard Linux cron format
  # "0 2 * * *" means: Minute 0, Hour 2 (2:00 AM), Every day, Every month, Every day of the week
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: busybox
            command: ["sh", "-c", "echo 'Running database backup...'; sleep 10; echo 'Backup complete!'"]
          restartPolicy: OnFailure
```

When you create this CronJob, it sits in the background. Every day at 2:00 AM, the CronJob automatically creates a new **Job**, which in turn creates a **Pod** to run the backup script.

### Useful Commands
- Check your CronJobs: `kubectl get cronjobs`
- Check the Jobs created by the CronJob: `kubectl get jobs`
- Check the Pods created by the Job: `kubectl get pods`

---

**Next →** [Network Policies](./06-network-policies.md)
