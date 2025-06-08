# Understanding Jobs and CronJobs

## 01. Job

- Jobs can be used to execute a workload and define when it completes.
- Typically, a Job will create one or more pods.
- After the Job is finished, the containers will exit and the pods will enter the _Completed_ status.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

In the above example, Here's a breakdown:

- **perl**: The Perl interpreter.
- **-Mbignum=bpi**: Imports the bignum module, specifically enabling transparent BigNumber support for pi calculations.
- **-wle**: Executes the one-line program that follows.
- **print bpi(2000)**: Prints the result of the bpi(2000) function to the console. The bpi() function calculates the number pi to the specified number of decimal places.
- **bpi(2000)**: The bpi() function calculates the number pi to the specified number of digits (in this case, 2000).
- The **backoffLimit** parameter means that, if it fails 4 times, this is the limit.
- All the Job does
- Although a normal pod is constantly running, when a Job is complete, it goes into the Completed status.
- This means that the container is no longer running, so the pod still exists, but the container is complete.

```
kubectl apply -f perl-job.yaml

kubectl get job
```

## 02. CronJob

- **CronJobs**, based on the capability of a Job, add value by allowing users to execute Jobs on a schedule.

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
              - /bin/sh
              - -c
              - date; echo Hello from the Kubernetes cluster
            restartPolicy: OnFailure
```

- To deploy the above yaml manifest:

```
kubectl apply -f hello-cronjob.yaml

kubectl get cronjob
```

- This cron job creates a few pods name hello, so we will use the following command to check the
  log of the Job:

```
kubectl get pods | grep hello

# To check the logs of these pods
kubectl logs hello-xxxx

# To delete cron jobs
kubectl delete cronjobs hello
```
