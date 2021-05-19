# K8S Learning


## Description

By Brian Yau

<br/>


Create Pod manifest:

```
kubectl run static-busybox --image=busybox --command sleep 1000 --restart=Never --dry-run=client -o yaml > static-busybox.yaml
```

<br/>

