# 02 - Scheduler 

- [02 - Scheduler](#02---scheduler)
  - [Manual Scheduling](#manual-scheduling)
  - [Labels & Selectors](#labels--selectors)
    - [Labels & Replicaset](#labels--replicaset)
    - [Annotations](#annotations)
    - [Taint](#taint)
  - [Node Selectors](#node-selectors)
  - [Node Affinity](#node-affinity)
    - [Node Affinity Types](#node-affinity-types)
  - [Taints and Tolerations vs. Node Affinity](#taints-and-tolerations-vs-node-affinity)
  - [Resoruce Requirements and Limits](#resoruce-requirements-and-limits)
    - [Resource Limits](#resource-limits)
    - [Default reosurce requirements and limits](#default-reosurce-requirements-and-limits)
  - [Daemon Sets](#daemon-sets)
  - [Static Pods](#static-pods)
    - [Static Pods vs. DaemonSets](#static-pods-vs-daemonsets)
  - [Multiple Schedulers](#multiple-schedulers)
    - [Deploy Additional Scheduler - kubeadm](#deploy-additional-scheduler---kubeadm)
    - [View Events](#view-events)
  - [Configuring Scheduler](#configuring-scheduler)

---

## Manual Scheduling

In a Pod manifest, if `nodeName` is not defined, **Scheduler** will see which node to schedule and bind the Pod to run in a specific node. Without the scheduler, if you create a Pod, it will go to the `Pending` state.

Note that the `nodeName` can only be specified in the creation time!

If you want to assign a running pod to another node, you can create a **Binding object** and do a POST request to the cluster API. Here is an example of pod-binding definition:

```
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

The POST request is like:
`curl --header "Content-Type: application/json" --request POST --data '{"apiVersion": "v1", "kind": "Binding", ...} http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/`

<br/>

---

## Labels & Selectors

To add labels for an object, use `metadata.labels`. For example:

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end

spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
```

<br/>

To use **selector** to filter out the labelled resources:

```
kubectl get pods --selector app=App1

kubectl get all --selector app=App1
```

<br/>

To add a label to a node:

```
kubectl label nodes <NODE_NAME> <k>=<v>
```

### Labels & Replicaset

In Replicaset, we commonly use **selector** to specify the `spec`:

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

<br/>

### Annotations

To record other metrics / information. It is a kind of metadata:

```
...

metadata:
  name: simple-webapp
  labels:
    app: App1
    fucntion: Front-end
  annotations:
    buildversion: 1.00
    owner: Joe
```

<br/>

### Taint

If you want to restrict pods from being placed to a specific node, you will use **taints**.

To taint a node:

```
kubectl taint nodes <NODE_NAME> app=App1:<TAINT_EFFECT>
```

There are 3 taint effect:
- `NoSchedule`: Reject
- `PreferNoSchedule`: Try to avoid placing
- `NoExecute`: Reject new pods and existing pods will be moved away

<br/>

To untaint:

```
kubectl taint nodes <NODE_NAME> app=App1:<TAINT_EFFECT>-
```

<br/>

Example `yaml`:

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

<br/>

Note that pods can be deployed in the Master node, but this is not a best practice.

```
kubectl describe node kubemaster | grep Taint
```

<br/>

---

## Node Selectors

To place resource-demanding pods to a specific node.

To limit a pod to run in a specific node:
1. Label the node: `kubectl laabel nodes <NODE_NAME> <LABEL_KEY>=<LABEL_VALUE>
2. Use `spec.nodeSelector.size` on Pod conf.

Example:
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large
```

<br/>

---

## Node Affinity

Using `nodeSelector` does not allow conditional statement like `OR`, `AND`.

Here is a equivalent configuration using node affinity:

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
```

`OR` condition:

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```

`Exclude`:
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn
            values:
            - Small
```

`Exist`:
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: Exists
```

<br/>

### Node Affinity Types

![picture 1](images/cd86e75e1aa48b64217a49d153c915945994e82f605367eda0980baa538bf6a6.png)  


- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution
- requiredDuringSchedulingRequiredDuringExecution

`DuringScheduling` means the pod does not exist and you are creating it for the first time. When using `requiredDuringScheduling`, you are critical about the placement of pods to be created, and the pod will only be created in the designated node as per the affinity rule.

However, if the placement of the pod is less important, you may set `preferredDuringScheduling` - priortize the affinity rule but if nothing match, the pod will be created in another node.

<br/>

`DuringExecution` means pods have been already running. `IgnoredDuringExcution` means any changes to the affinity rule will not make any impact to the existing pods.

<br/>

`RequiredDuringExecution` means the existing pods not obeying the affinity rule will be removed.

<br/>

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: red
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: red
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
```

<br/>

---

## Taints and Tolerations vs. Node Affinity

![picture 2](images/88b23fcf746b51f4bb58cd552916e8403c25237d166a071fea4c611bda005667.png)  

<br/>

![picture 3](images/2ddba53ec78a6f81ea7906e9d3d27484b632c94911e56f7c8a6ffb618f2efc8a.png)  

<br/>

![picture 4](images/96174ab0d9e11d4aeffdca69dab473925c806dcf14923b8a268b4b6f15c1cb26.png)  

<br/>

---

## Resoruce Requirements and Limits

 Resource request of a container by default:
 - CPU: 0.5 
 - Memory: 256 MB

Note:
1 cpu = 1 AWS vCPU / 1 GCP Core / 1 Azure Core / 1 Hyperthread

To specify the desired resource request, declare `spec.container.resources.requests`:

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
```

Note:
1G = 1,000,000,000 bytes
1Gi = 1,073,741,824 bytes
1M = 1,000,000 bytes
1Mi = 1,048,576 bytes
1K = 1,000 bytes
1Ki = 1,024 bytes

<br/>


### Resource Limits

- By default limit = 1 vCPU + 512 Mi

To specify the desired resource limit, declare `spec.container.resources.limits`:

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```

<br/>

When the container exceeds CPU limits, **CPU throttle** happens.

When the container exceeds memory usage **constantly**, it will get terminated. (Note when the usage is higher than the limit, it will not get terminated immediately - only when it happens **constantly**)

<br/>

### Default reosurce requirements and limits

For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```

- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/

- https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource

<br/>

---

## Daemon Sets

![picture 3](images/ac08798428e49bbd9970032455c5ed8ad634a5838b8d9d9dcb9d4c3f2ffdf9f6.png)  

![picture 4](images/5227fa61eb11b13f88d2269a8ca6c966ec5e5e37b1a91523779292cb5edc39b2.png)  

![picture 5](images/2e390ebe63270fdc9dfa45b687f99ce3b1d015e99f21b96e6b855c2ef3c78842.png)  

Common use cases:

- Kube-proxy
- weave-net

<br/>

Recall the replicaset Definition:

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
      spec:
        containers:
        - name: monitoring-agent
          image: monitoring-agent
```

Deamon set definition is very similar:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent 
```

<br/>

To create a daemon set:

```
kubectl create -f daemon-set-definition.yaml
```

<br/>

To view:

```
kubectl get daemonsets

kubectl get ds --all-namespaces

kubectl describe daemonsets <DAEMONSET_NAME>
```

<br/>

---

## Static Pods

**Static Pods** are managed directly by the kubelet daemon on a specific node, **without the API server observing them**. 
- Whereas most Pods are managed by the control plane (for example, a Deployment), for static Pods, the kubelet directly supervises each static Pod (and restarts it if it fails).
- Static Pods are always bound to one Kubelet on a specific node. 
- The main use for static Pods is to run a self-hosted control plane: in other words, using the kubelet to supervise the individual control plane components.
- The kubelet automatically tries to create a mirror Pod on the Kubernetes API server for each static Pod. This means that the Pods running on a node are visible on the API server, but cannot be controlled from there.

<br/>

kubelet will monitor the configuration files in the path `/etc/kubernetes/manifests`. When the manifests are edited / created, it kills the existing pods and redeploy / deploy accordingly.

- This can be changed if you specify the `pod-manifest-path` when starting `kubelet`:

```kubelet.service
ExecStart=/usr/local/bin/kubelet \\
  --pod-manifest-path=/etc/Kubernetes/manifests
```

- Alternatively, you can specify `--config=kubeconfig.yaml` in the `kubelet.service` file, and then :

```kubeconfig.yaml
staticPodPath: /etc/kubernetes/manifests
```

<br/>

To view the static pods created:

```
docker ps
```

![picture 6](images/601bc07409047e4b79a24272f511a2f97b59db0fc469a897fc2b773905d195b0.png)  

<br/>

If the Master node presents, it can also see the static pod created by kubelet; however it cannot edit / remove it.

Also, you will see the pod name has `-<NODENAME>` appended.

<br/>

### Static Pods vs. DaemonSets

![picture 7](images/06b27a98290ff329e06d577fe98d28de163ef095ac9165a0c94ddddba783e3f8.png)  

<br/>

---

## Multiple Schedulers

You can write your own packaged Schedulers in Kubernetes.

K8S supports multiple Schedulers at the same time. To run one more scheduler, declare it in a service file:

```kube-scheduler-custom.service
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --scheduler-name=kube-scheduler-custom
```

<br/>

### Deploy Additional Scheduler - kubeadm

`/etc/kubernetes/manifests/kube-scheduler.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
    - command:
      - kube-scheduler
      - --address=127.0.0.1
      - --kubeconfig=/etc/kubernetes/scheduler.conf
      - --leader-elect=true
      image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
      name: kube-scheduler
```

`kube-scheduler-custom.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: kube-scheduler-custom
  namespace: kube-system
spec:
  containers:
    - command:
      - kube-scheduler
      - --address=127.0.0.1
      - --kubeconfig=/etc/kubernetes/scheduler.conf
      - --leader-elect=true
      - --scheduler-name=kube-scheduler-custom
      - --lock-object-name=kube-schedular-custom
      image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
      name: kube-scheduler
```

<br/>

Once you have the new kube scheduler service running, you will see it in the kube-system namespace:

```
kubectl get pods --namespace=kube-system
```

<br/>

To use this scheduler, in pod manifest:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: kube-scheduler-custom
```

<br/>

### View Events

```
kubectl get events

kubectl logs kube-scheduler-custom --namespace=kube-system
```

<br/>

---

## Configuring Scheduler

![picture 8](images/66390d071ac9be059ce943a3ed4e73818d98bb9b96459e767e602c3b98b749df.png)  

![picture 9](images/054c85e390d0af7a1a9318d2d24c4070dda23d3a7a74a6984992feef6af03c38.png)  

<br/>

---