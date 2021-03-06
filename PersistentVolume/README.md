## PersistentVolume example

For the examples here, we will not create a `PersistentVolume` this will be created automatically when you create the `PersistentVolumeClaim` because the `PersistentVolumeClaim` will request a type of storage that is declared on the `StorageClass`.

- `PersistentVolume` (PV) is the "physical" disk itself.

- `PersistentVolumeClaim` (PVC) is a request/claim to use a "physical" disk.

- `StorageClass` allow you to create different type of storage, "classes" of storage, to be requested/claimed by the PersistentVolumeClaim (PVC).

####- First we create a `StorageClass`:
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

####- Second we create a `PersistentVolumeClaim`:
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

Notice that we are using the **annotations** `volume.beta.kubernetes.io/storage-class: "generic"` to point to the **StorageClass** we created above, this will define the `type` and `zone` for the disk physical (PV) that will be created.

######Check your PersistentVolumeClaim:
```
$ kubectl get PersistentVolumeClaim
NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
html-disk   Bound     pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8   10Gi       RWO           45s
```
######Few seconds later a PersistentVolume (PV) will be created, check it:
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
- The `PDName:     gke-multi1-5-2-207a043-pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8` is the name of the disk created on the Google Cloud Platform, you can check it with below command:

```
$ gcloud compute disks list --filter NAME = gke-multi1-5-2-207a043-pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8
NAME                                                             ZONE            SIZE_GB  TYPE         STATUS
gke-multi1-5-2-207a043-pvc-b6bfda50-eafa-11e6-9eb1-42010a8400e8  europe-west1-c  10       pd-standard  READY
```
- With the above output you can see that as per the `StorageClass` we created above the disk is `type: pd-standard` and is in the `zone: europe-west1-c`.

####- Now let's use the Persistent Volume:

```
kind: Pod
apiVersion: v1
metadata:
  name: pod-webserver
spec:
  containers:
    - name: my-web-server
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: html-disk
```
- To use the disk you need to tell where you want to mount it using the `volumeMounts` declaration followed by `mountPath: "/var/www/html"` and the name of the `volumes` you want to mount using `name: data`.
- After that you declare the `volumes` and give a name to it `name: data`, you need to specify which `persistentVolumeClaim` you want to use, for that you `claimName: html-disk` declaration.

######Let's check the volume on the POD:
```
$ kubectl get po
NAME            READY     STATUS    RESTARTS   AGE
pod-webserver   1/1       Running   0          58s

$ kubectl exec -ti pod-webserver bash
root@pod-webserver:/# 

root@pod-webserver:/# df -h                                                                                                                                                                                             
Filesystem      Size  Used Avail Use% Mounted on
overlay          95G  3.3G   92G   4% /
tmpfs           1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1        95G  3.3G   92G   4% /etc/hosts
/dev/sdb        9.8G   37M  9.3G   1% /var/www/html
tmpfs           1.9G   12K  1.9G   1% /run/secrets/kubernetes.io/serviceaccount
shm              64M     0   64M   0% /dev/shm
```
- We can see that there is a new disk `/dev/sdb` that is mounted on `/var/www/html`.

# Caveats

- Currently there is no way to declare a `StorageClass` for multiple zones.
- If you use the Alpha annotation `volume.alpha.kubernetes.io/storage-class: generic` on your `PersistentVolumeClaim` the `StorageClass` will be ignored, you can use the Alpha annotation if you want to create disks in multiple zones, but you will not be able to specify the `type` of the disk and a `pd-standard` will be created.
