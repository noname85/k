## PersistentVolume example

- PersistentVolume (PV) is the "physical" disk itself.
- PersistentVolumeClaim (PVC) is a request/claim to use a "physical" disk.
- StorageClass allow you to create different type of storage, "classes" of storage, to be requested/claimed.

For the examples here, we will not create a "PersistentVolume" this will be created automatically when you create the "PersistentVolumeClaim" because the "PersistentVolumeClaim" will request a type of storage that is declared on the "StorageClass".

####- First we create a "StorageClass":
```
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: generic
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  zone: europe-west1-c
```

- **`type`**: pd-standard or pd-ssd. Default: pd-ssd
- **`zone`**: GCE zone. If not specified, a random zone in the same region as controller-manager will be chosen.

######Check your StorageClass:
```
$ kubectl get StorageClass
NAME      TYPE
generic   kubernetes.io/gce-pd   
```

####- Second we create a "PersistentVolumeClaim":
```
kind: PersistentVolumeClaim 
apiVersion: v1 
metadata: 
  name: html-disk
  annotations: 
    volume.beta.kubernetes.io/storage-class: "generic" 
spec: 
  accessModes: 
    - "ReadWriteOnce" 
  resources: 
    requests: 
      storage: "10Gi"
```

notice that we are using the **annotations** `volume.beta.kubernetes.io/storage-class: "generic"` to point to the **StorageClass** we created above, this will define the `type` and `zone` for the disk

######Check your PersistentVolumeClaim:
```
$ kubectl get PersistentVolumeClaim
NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
html-disk   Bound     pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8   10Gi       RWO           45s
```
######Few seconds later a PersistentVolume will be created, check it:
```
$ kubectl get PersistentVolume
NAME                                       CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM               REASON    AGE
pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8   10Gi       RWO           Delete          Bound     default/html-disk             54s
```
