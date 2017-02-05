## PersistentVolume example

- PersistentVolume (PV) is the "physical" disk itself.
- PersistentVolumeClaim (PVC) is a request/claim to use a "physical" disk.
- StorageClass allow you to create different type of storage, "classes" of storage, to be requested/claimed by the PersistentVolumeClaim (PVC).

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
######If you "describe" your PersistentVolume:
```
$ kubectl describe PersistentVolume pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8
Name:           pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8
Labels:         failure-domain.beta.kubernetes.io/region=europe-west1
                failure-domain.beta.kubernetes.io/zone=europe-west1-c
StorageClass:   generic
Status:         Bound
Claim:          default/html-disk
Reclaim Policy: Delete
Access Modes:   RWO
Capacity:       10Gi
Message:
Source:
    Type:       GCEPersistentDisk (a Persistent Disk resource in Google Compute Engine)
    PDName:     gke-multi1-5-2-207a043-pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8
    FSType:
    Partition:  0
    ReadOnly:   false
No events.
```
- Notice that the `StorageClass:   generic` is the one declared on the PersistentVolumeClaim (PVC) using the **annotation** `volume.beta.kubernetes.io/storage-class: "generic" `.
- The `PDName:     gke-multi1-5-2-207a043-pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8` is the name of the disk created on the GCP you can check it with below command:

```
$ gcloud compute disks list --filter NAME = gke-multi1-5-2-207a043-pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8
NAME                                                             ZONE            SIZE_GB  TYPE         STATUS
gke-multi1-5-2-207a043-pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8  europe-west1-c  10       pd-standard  READY
```
- With the above output you can see that as per the `StorageClass` we created above the disk is `type: pd-standard` and is in the `zone: europe-west1-c`.
