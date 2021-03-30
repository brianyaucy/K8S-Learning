# 02 - Scheduler 

- [02 - Scheduler](#02---scheduler)
  - [Manual Scheduling](#manual-scheduling)

---

## Manual Scheduling

In a Pod manifest, if `nodeName` is not defined, **Scheduler** will see which node to schedule and bind the Pod to run in a specific node.

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

