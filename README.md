# Kubernetes Volumes & Persistent Storage Lab

## Overview

This lab demonstrates how storage works in Kubernetes using different volume types.
The objective is to understand how containers share data, how node storage can be mounted, and how persistent storage is managed using PersistentVolumes (PV) and PersistentVolumeClaims (PVC).

The lab covers the following Kubernetes storage concepts:

* emptyDir Volume
* hostPath Volume
* PersistentVolume (PV)
* PersistentVolumeClaim (PVC)
* Multi-volume Pods

---

# Lab Objectives

By completing this lab you will learn how to:

* Share data between containers in the same Pod
* Mount node storage into a Pod
* Create and manage PersistentVolumes
* Bind PersistentVolumeClaims to PersistentVolumes
* Use persistent storage inside Pods
* Configure Pods with multiple volumes

---

# Lab Structure

The lab consists of **3 main parts and 6 tasks**.

| Part   | Topic                    | Tasks              |
| ------ | ------------------------ | ------------------ |
| Part 1 | Ephemeral & Node Storage | emptyDir, hostPath |
| Part 2 | Persistent Storage       | PV, PVC            |
| Part 3 | Advanced Pod Storage     | Multi-volume Pod   |

---

# Part 1 — emptyDir & hostPath

## Task 1 — emptyDir Volume

Create a Pod with two containers sharing the same volume.

**Concept**

`emptyDir` allows containers in the same Pod to share files.

The volume exists only while the Pod is running.

### Steps

Apply the configuration

```
kubectl apply -f 01-emptydir-volume.yaml
```

Check Pod logs

```
kubectl logs emptydir-demo -c reader
```

Verify the file exists

```
kubectl exec emptydir-demo -c writer -- cat /shared/log.txt
```

Delete and recreate the Pod

```
kubectl delete pod emptydir-demo
kubectl apply -f 01-emptydir-volume.yaml
```

Result:
The data is deleted because `emptyDir` is temporary.

---

## Task 2 — hostPath Volume

Mount a directory from the Kubernetes node into the Pod.

### Create folder on the node

```
minikube ssh -- mkdir -p /tmp/webdata
```

Create an HTML file

```
minikube ssh -- 'echo "<h1>From the Node!</h1>" > /tmp/webdata/index.html'
```

Deploy the Pod

```
kubectl apply -f 02-hostpath-volume.yaml
```

Test from inside the Pod

```
kubectl exec hostpath-demo -- curl localhost
```

Expected output:

```
<h1>From the Node!</h1>
```

Update the HTML file without restarting the Pod

```
minikube ssh -- 'echo "<h1>Updated!</h1>" > /tmp/webdata/index.html'
```

---

# Part 2 — Persistent Storage

## Task 3 — Create PersistentVolume

A PersistentVolume provides storage resources to the cluster.

Apply the configuration:

```
kubectl apply -f 03-persistentvolume.yaml
```

Check the status:

```
kubectl get pv
```

---

## Task 4 — Create PersistentVolumeClaim

A PVC requests storage from available PersistentVolumes.

Apply the claim

```
kubectl apply -f 04-persistentvolumeclaim.yaml
```

Verify the binding

```
kubectl get pv,pvc
```

Expected result:

```
STATUS: Bound
```

---

## Task 5 — Use PVC in a Pod

Deploy a Pod using the created PVC.

```
kubectl apply -f 05-pod-with-pvc.yaml
```

Create a file in the mounted directory

```
kubectl exec pvc-demo -- sh -c "echo persistent > /usr/share/nginx/html/test.txt"
```

Delete the Pod

```
kubectl delete pod pvc-demo
```

Recreate the Pod

```
kubectl apply -f 05-pod-with-pvc.yaml
```

Check if the file still exists

```
kubectl exec pvc-demo -- cat /usr/share/nginx/html/test.txt
```

Result:
The file still exists because the storage is **persistent**.

---

# Part 3 — Multi-Volume Pod

This task demonstrates how a Pod can use multiple volume types simultaneously.

The Pod will use:

| Volume     | Type              | Mount Path  |
| ---------- | ----------------- | ----------- |
| cache-vol  | emptyDir (Memory) | /tmp/cache  |
| data-vol   | PVC               | /data       |
| config-vol | ConfigMap         | /etc/config |

---

## Create required resources

Create PVC

```
kubectl apply -f 06-deployment-with-pvc.yaml
```

Create ConfigMap

```
kubectl create configmap app-config --from-literal=KEY=value
```

Deploy the Pod

```
kubectl apply -f 07-multi-volume-pod.yaml
```

Verify mounts

```
kubectl exec multi-volume-demo -- df -h
```

Check ConfigMap

```
kubectl exec multi-volume-demo -- ls /etc/config
```

---

# Key Kubernetes Storage Concepts

### emptyDir

Temporary storage shared between containers in the same Pod.

### hostPath

Mounts a directory from the Kubernetes node into the Pod.

### PersistentVolume (PV)

Cluster-level storage resource.

### PersistentVolumeClaim (PVC)

A request for storage by a Pod.

### ConfigMap Volume

Allows configuration data to be injected into containers.

---

# Repository Structure

```
k8s-volumes-lab/
│
├── 01-emptydir-volume.yaml
├── 02-hostpath-volume.yaml
├── 03-persistentvolume.yaml
├── 04-persistentvolumeclaim.yaml
├── 05-pod-with-pvc.yaml
├── 06-deployment-with-pvc.yaml
├── 07-multi-volume-pod.yaml
│
└── README.md
```

---

# Conclusion

This lab demonstrates essential Kubernetes storage mechanisms including ephemeral volumes, node-mounted storage, and persistent storage using PV and PVC.

Understanding these concepts is crucial for deploying stateful applications in Kubernetes environments.

---

# Author

Mariam Mansour
DevOps Engineer
