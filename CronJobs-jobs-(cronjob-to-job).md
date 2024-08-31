
#   Cronjobs , jobs , port-forword and create job from existing cronJob

- Cronjobs creation

```

root@ip-192-168-1-177:~# ls
abc.log  aws  awscliv2.zip  hello-deploy.yaml  k8s_RBAC  snap
root@ip-192-168-1-177:~# kubectl apply -f hello-deploy.yaml
deployment.apps/helloweb created
root@ip-192-168-1-177:~# kubectl get po
NAME                        READY   STATUS    RESTARTS   AGE
helloweb-68f7f55f67-fpbh4   1/1     Running   0          16s
helloweb-68f7f55f67-k8g4v   1/1     Running   0          16s
helloweb-68f7f55f67-m8tm8   1/1     Running   0          16s
root@ip-192-168-1-177:~#  cat hello-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloweb
  labels:
    app: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
      tier: web
  template:
    metadata:
      labels:
        app: hello
        tier: web
    spec:
      containers:
      - name: hello-app
        image: us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 200m
root@ip-192-168-1-177:~#


root@ip-192-168-1-177:~# kubectl get deploy -o wide
NAME       READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                          SELECTOR
helloweb   3/3     3            3           81s   hello-app    us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0   app=hello,tier=web
root@ip-192-168-1-177:~# kubectl get po -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP               NODE                             NOMINATED NODE   READINESS GATES
helloweb-68f7f55f67-fpbh4   1/1     Running   0          93s   192.168.61.10    ip-192-168-50-28.ec2.internal    <none>           <none>
helloweb-68f7f55f67-k8g4v   1/1     Running   0          93s   192.168.24.128   ip-192-168-28-112.ec2.internal   <none>           <none>
helloweb-68f7f55f67-m8tm8   1/1     Running   0          93s   192.168.15.190   ip-192-168-28-112.ec2.internal   <none>           <none>
root@ip-192-168-1-177:~# kubectl get no  -o wide
NAME                             STATUS   ROLES    AGE   VERSION               INTERNAL-IP      EXTERNAL-IP      OS-IMAGE         KERNEL-VERSION                 CONTAINER-RUNTIME
ip-192-168-28-112.ec2.internal   Ready    <none>   17m   v1.22.9-eks-810597c   192.168.28.112   54.175.112.152   Amazon Linux 2   5.4.196-108.356.amzn2.x86_64   docker://20.10.13
ip-192-168-50-28.ec2.internal    Ready    <none>   17m   v1.22.9-eks-810597c   192.168.50.28    44.198.157.214   Amazon Linux 2   5.4.196-108.356.amzn2.x86_64   docker://20.10.13
root@ip-192-168-1-177:~#


```
#  port forward  option
```
root@ip-192-168-1-177:~# kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
helloweb   3/3     3            3           3m57s
root@ip-192-168-1-177:~#  kubectl port-forward deploy/helloweb 9089:8080
Forwarding from 127.0.0.1:9089 -> 8080
Forwarding from [::1]:9089 -> 8080
^Croot@ip-192-168-1-177:~#
root@ip-192-168-1-177:~#
root@ip-192-168-1-177

```

-  CronJob in kubernites

```
root@ip-192-168-1-177:~# kubectl get cronJob
NAME         SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello-cron   */2 * * * *   False     0        65s             80s
root@ip-192-168-1-177:~# kubectl delete cronJob hello-cron
cronjob.batch "hello-cron" deleted
root@ip-192-168-1-177:~# cat cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello-cron
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
root@ip-192-168-1-177:~# kubectl apply -f cronjob.yaml
cronjob.batch/hello-cron created
root@ip-192-168-1-177:~# kubectl get CronJob
NAME         SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello-cron   */2 * * * *   False     0        22s             40s
root@ip-192-168-1-177:~# kubectl get po -A
NAMESPACE     NAME                        READY   STATUS      RESTARTS   AGE
default       hello-cron-27619592-w7vj6   0/1     Completed   0          34s
default       helloweb-68f7f55f67-fpbh4   1/1     Running     0          32m
default       helloweb-68f7f55f67-k8g4v   1/1     Running     0          32m
default       helloweb-68f7f55f67-m8tm8   1/1     Running     0          32m
kube-system   aws-node-7s7jr              1/1     Running     0          49m
kube-system   aws-node-fqxvv              1/1     Running     0          49m
kube-system   coredns-7f5998f4c-b4nxl     1/1     Running     0          57m
kube-system   coredns-7f5998f4c-pm82k     1/1     Running     0          57m
kube-system   kube-proxy-brqqw            1/1     Running     0          49m
kube-system   kube-proxy-zqdwl            1/1     Running     0          49m
root@ip-192-168-1-177:~# kubectl logs hello-cron-27619592-w7vj6
Thu Jul  7 06:32:00 UTC 2022
Hello from the Kubernetes cluster
root@ip-192-168-1-177:~#
root@ip-192-168-1-177:~# kubectl get cj
NAME         SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello-cron   */2 * * * *   False     0        6s              4m24s
root@ip-192-168-1-177:~#

```


###   Create jobs

```
root@ip-192-168-1-177:~# vi jobs.yaml
root@ip-192-168-1-177:~# kubectl apply -f jobs.yaml
job.batch/job-prog created
root@ip-192-168-1-177:~# kubectl get jobs
NAME                  COMPLETIONS   DURATION   AGE
hello-cron-27619596   1/1           3s         5m13s
hello-cron-27619598   1/1           3s         3m13s
hello-cron-27619600   1/1           3s         73s
job-prog              1/1           3s         9s
root@ip-192-168-1-177:~# kubectl get  po -A
NAMESPACE     NAME                        READY   STATUS      RESTARTS   AGE
default       hello-cron-27619596-4ggrp   0/1     Completed   0          5m30s
default       hello-cron-27619598-dz9rt   0/1     Completed   0          3m30s
default       hello-cron-27619600-x8jls   0/1     Completed   0          90s
default       helloweb-68f7f55f67-fpbh4   1/1     Running     0          41m
default       helloweb-68f7f55f67-k8g4v   1/1     Running     0          41m
default       helloweb-68f7f55f67-m8tm8   1/1     Running     0          41m
default       job-prog-cfd2s              0/1     Completed   0          26s
kube-system   aws-node-7s7jr              1/1     Running     0          58m
kube-system   aws-node-fqxvv              1/1     Running     0          58m
kube-system   coredns-7f5998f4c-b4nxl     1/1     Running     0          66m
kube-system   coredns-7f5998f4c-pm82k     1/1     Running     0          66m
kube-system   kube-proxy-brqqw            1/1     Running     0          58m
kube-system   kube-proxy-zqdwl            1/1     Running     0          58m
root@ip-192-168-1-177:~# cat jobs.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-prog
spec:
  template:
        spec:
          containers:
          - name: job-prog
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
root@ip-192-168-1-177:~#
root@ip-192-168-1-177:~# kubectl get  po -A -w
NAMESPACE     NAME                        READY   STATUS      RESTARTS   AGE
default       hello-cron-27619598-dz9rt   0/1     Completed   0          4m29s
default       hello-cron-27619600-x8jls   0/1     Completed   0          2m29s
default       hello-cron-27619602-zbqp8   0/1     Completed   0          29s
default       helloweb-68f7f55f67-fpbh4   1/1     Running     0          42m
default       helloweb-68f7f55f67-k8g4v   1/1     Running     0          42m
default       helloweb-68f7f55f67-m8tm8   1/1     Running     0          42m
default       job-prog-cfd2s              0/1     Completed   0          85s
kube-system   aws-node-7s7jr              1/1     Running     0          59m
kube-system   aws-node-fqxvv              1/1     Running     0          58m
kube-system   coredns-7f5998f4c-b4nxl     1/1     Running     0          67m
kube-system   coredns-7f5998f4c-pm82k     1/1     Running     0          67m
kube-system   kube-proxy-brqqw            1/1     Running     0          58m
kube-system   kube-proxy-zqdwl            1/1     Running     0          59m
^Croot@ip-192-168-1-177:~#
root@ip-192-168-1-177:~# kubectl logs job-prog-cfd2s
Thu Jul  7 06:41:05 UTC 2022
Hello from the Kubernetes cluster
root@ip-192-168-1-177:~#
```

###  Create job from CronJob

```
root@ip-192-168-1-177:~# kubectl create job cronjob-to-jobs --from=cronjob/hello-cron --context i-0c57f390bbb64b651@sambasiva.us-east-1.eksctl.io
job.batch/cronjob-to-jobs created
root@ip-192-168-1-177:~#


root@ip-192-168-1-177:~# kubectl get jobs
NAME                  COMPLETIONS   DURATION   AGE
cronjob-to-jobs       1/1           4s         23s
hello-cron-27619604   1/1           3s         4m10s
hello-cron-27619606   1/1           3s         2m10s
hello-cron-27619608   1/1           3s         10s
job-prog              1/1           3s         7m6s
root@ip-192-168-1-177:~#


root@ip-192-168-1-177:~# kubectl  get no  -o wide
NAME                             STATUS   ROLES    AGE   VERSION               INTERNAL-IP      EXTERNAL-IP      OS-IMAGE         KERNEL-VERSION                 CONTAINER-RUNTIME
ip-192-168-28-112.ec2.internal   Ready    <none>   67m   v1.22.9-eks-810597c   192.168.28.112   54.175.112.152   Amazon Linux 2   5.4.196-108.356.amzn2.x86_64   docker://20.10.13
ip-192-168-50-28.ec2.internal    Ready    <none>   67m   v1.22.9-eks-810597c   192.168.50.28    44.198.157.214   Amazon Linux 2   5.4.196-108.356.amzn2.x86_64   docker://20.10.13
root@ip-192-168-1-177:~#


root@ip-192-168-1-177:~# kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
helloweb   3/3     3            3           49m
root@ip-192-168-1-177:~# kubectl port-forward
error: TYPE/NAME and list of ports are required for port-forward
See 'kubectl port-forward -h' for help and examples
root@ip-192-168-1-177:~# kubectl port-forward  deploy/helloweb  9087:8080
Forwarding from 127.0.0.1:9087 -> 8080
Forwarding from [::1]:9087 -> 8080


```
