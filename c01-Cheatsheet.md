# Cheatsheet

- [Cheatsheet](#cheatsheet)
  - [Common commands](#common-commands)
    - [Create an NGINX Pod](#create-an-nginx-pod)
    - [Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)](#generate-pod-manifest-yaml-file--o-yaml-dont-create-it--dry-run)
    - [Create a deployment](#create-a-deployment)
    - [Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)](#generate-deployment-yaml-file--o-yaml-dont-create-it--dry-run)
    - [Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)](#generate-deployment-yaml-file--o-yaml-dont-create-it--dry-run-with-4-replicas---replicas4)

---

## Common commands


### Create an NGINX Pod

```
kubectl run nginx --image=nginx
```

<br/>

### Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

<br/>

### Create a deployment

```
kubectl create deployment --image=nginx nginx
```

<br/>

### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

<br/>

### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

- Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

<br/>

---