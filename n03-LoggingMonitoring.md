# 03 - Logging & Monitoring

- [03 - Logging & Monitoring](#03---logging--monitoring)
  - [Monitor Cluster Components](#monitor-cluster-components)
    - [Heapster vs. Metrics Server](#heapster-vs-metrics-server)

---

## Monitor Cluster Components

![picture 10](images/e1e9b03b35794ee67dbb2abeefbe4326b78e574536955f40cba276f531a52352.png)  

<br/>

### Heapster vs. Metrics Server

- Heapster (deprated)

- Metrics Server
  - In memory
  - No hisotrical metrics!

<br/>

Kubelet has a **cAdvisor** for getting the metrics in Pods and expose it via `kubectl`.

<br/>

To get Metrics Server started:

```
minikube addons enable metrics-server
```

OR

```
git clone https://github.com/kubernetes-incubator/metrics-server
kubectl create -f deploy/1.8+/
```

<br/>

To view metrics:

```
kubectl top node

kubectl top pod
```

<br/>

---

