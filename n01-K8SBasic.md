# 01 - K8S Basic

- [01 - K8S Basic](#01---k8s-basic)
  - [Basic Kubectl - Pods](#basic-kubectl---pods)
  - [Manifest](#manifest)
  - [ReplicationController](#replicationcontroller)
  - [ReplicaSets](#replicasets)

----

## Basic Kubectl - Pods

To list running pods:

```
kubectl get pods
```

<br/>

To run a pod named `nginx` using the image `nginx` from Dockerhub:

```
kubectl run nginx --image=nginx
```

<br/>

To get the configuration about a pod:

```
kubectl describe pod <POD_NAME>

kubectl describe pod <POD_NAME> | grep -i image
```

To get the configuration in `YAML` format:

```
kubectl get pod <POD_NAME> -o yaml

kubectl get pod <POD_NAME> -o yaml | grep -i image
```

<br/>

To show more information about the running pods in table format:

```
kubectl get pods -o wide
```

Alternatively you can check the pod detail:

```
kubectl get pod <POD_NAME> -o yaml | grep -i nodeName
```

<br/>

To delete a pod:

```
kubectl delete pod <POD_NAME>
```

<br/>

---

## Manifest

To create a skeleton `YAML` file:

```
kubectl run <POD_NAME> --image=<IMAGE_NAME> --dry-run=client -o yaml > pod.yaml
```

The output will be something like this:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis123
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

<br/>

To create a pod using a fanifest:

```
kubectl create -f <MANIFEST.yaml>

OR

kubectl apply -f <MANIFEST.yaml>
```

<br/>

To edit an existing pod's configuration:

```
kubectl edit pod <POD_NAME>
```

After editing, K8S will try to re-run using the new configuration.

<br/>

---

## ReplicationController

Used for resilience / ensure the desired state.

<br/>

**ReplicationController**

Skeleton:

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    <POD_CONFIG>
  replicas: <NUMBER>
```

Example:

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadatta:  
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```

<br/>

To create a replica set using a `YAML`:

```
kubectl create -f <RC-CONFIG.yaml>
```

<br/>

To view running replicationcontroller:

```
kubectl get replicationcontroller
```

<br/>

You can also see the running pods created by the replication controller:

```
kubectl get pods
```

<br/>

---

## ReplicaSets

ReplicaSet skeleton (major difference: `spec.selector`)

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    <POD_CONFIG>
  replicas: <NUMBER>
  selector: 
    matchLabels:
      type: front-end
```

<br/>

Example:

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadatta:  
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector: 
    matchLabels:
      type: front-end
```

<br/>

To run a ReplicaSet using `kubectl`:

```
kubectl create -f <ReplicaSet.yaml>
```

<br/>

```
kubectl get replicaset
```

<br/>

```
kubectl get pods
```

<br/>

Note **ReplicaSet** uses the `label` filter to monitor the pods.

To scale:

- Method 1: Change the `replica` number in the manifest and run 
  - `kubectl replace -f <manifest.yml>

- Method 2: Use the `scale` command with `--replicas` option
  - `kubectl scale --replicas=6 -f <manifest.yml>`
  - `kubectl scale --replicas=6 replicaset <REPLICASET_NAME>`

<br/>

