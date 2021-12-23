---
title: Multiple Parallel Scanners
taxonomy:
    category: docs
---

### Increase Scanner Scalability with Multiple Scanners
To increase scanner performance and scalability, NeuVector supports deploying multiple scanner pods which can, in parallel, scan images in registries. The controller assigns scanning tasks to each available scanner pod. Scanner pods can be scaled up or down easily as needed using Kubernetes.

Scanner pods should be deployed to separate nodes to spread the workload across different host resources. Remember that a scanner requires enough memory to pull and expand the image, so it should have available to it more than the largest image size to be scanned. If necessary, scanners can be placed on specific nodes or avoid placing multiple pods on one node using standard Kubernetes node labels, taints/tolerations, or node affinity configurations.

#### Deploying Multiple Scanners
Create a new role binding
```
kubectl create rolebinding neuvector-admin --clusterrole=admin --serviceaccount=neuvector:default -n neuvector
```

Or for OpenShift
```
oc adm policy add-role-to-user admin system:serviceaccount:neuvector:default -n neuvector
```

Use the file below to deploy multiple scanners. Edit the replicas to increase or decrease the number of scanners running in parallel.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: neuvector-scanner-pod
  namespace: neuvector
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  replicas: 2
  selector:
    matchLabels:
      app: neuvector-scanner-pod
  template:
    metadata:
      labels:
        app: neuvector-scanner-pod
    spec:
      imagePullSecrets:
        - name: regsecret
      containers:
        - name: neuvector-scanner-pod
          image: neuvector/scanner
          imagePullPolicy: Always
          env:
            - name: CLUSTER_JOIN_ADDR
              value: neuvector-svc-controller.neuvector
      restartPolicy: Always
```

Create or update the CVE database updater cron job.
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: neuvector-updater-pod
  namespace: neuvector
spec:
  schedule: "0 0  * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: neuvector-updater-pod
        spec:
          imagePullSecrets:
          - name: regsecret
          containers:
          - name: neuvector-updater-pod
            image: neuvector/updater
            imagePullPolicy: Always
            lifecycle:
              postStart:
                exec:
                  command:
                  - /bin/sh
                  - -c
                  - TOKEN=`cat /var/run/secrets/kubernetes.io/serviceaccount/token`; /usr/bin/curl -kv -X PATCH -H "Authorization:Bearer $TOKEN" -H "Content-Type:application/strategic-merge-patch+json" -d '{"spec":{"template":{"metadata":{"annotations":{"kubectl.kubernetes.io/restartedAt":"'`date +%Y-%m-%dT%H:%M:%S%z`'"}}}}}' 'https://kubernetes.default/apis/apps/v1/namespaces/neuvector/deployments/neuvector-scanner-pod'
            env:
              - name: CLUSTER_JOIN_ADDR
                value: neuvector-svc-controller.neuvector
          restartPolicy: Never
```

Please note that in initial releases the presence and status of multiple scanners is only visible in Kubernetes with 'kubectl get pods -n neuvector' and will not be displayed in the web console. 

Results from all scanners are shown in the Assets -> Registries menu. Additional scanner monitoring features will be added in future releases.

#### Operations and Debugging
Each scanner pod will query the registries to be scanned to pull down the complete list of available images and other data. Each scanner will then be assigned an image to pull and scan from the registry.

To inspect the scanner behavior, logs from each scanner pod can be examined using
```
kubectl logs <scanner-pod-name> -n neuvector
```

### Performance Planning
Experiment with varying numbers of scanners on registries with a large number of images to observe the scan completion time behavior in your environment. 2-5 scanners as the replica setting should be sufficient for most cases.

When a scan task is assigned to a scanner, it pulls the image from the registry (after querying the registry for the list of available images). The amount of time it takes to pull the image (download) typically consumes the most time. Multiple scanners can be pulling images from the same registry in parallel, so the performance may be limited by registry or network bandwidth.

Large images will take more time to pull as well as need to be expanded to scan them, consuming more memory. Make sure each scanner has enough memory allocated to it to handle more than the largest expected image (10% more minimum).

Multiple scanner pods can be deployed to the same host/node, but considerations should be made to ensure the host has enough memory, CPU, and network bandwidth for maximizing scanner performance.