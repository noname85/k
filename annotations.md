###This is a list of the annotations used in these examples.

######- Create an Ingress in GCP using an already created Static IP address:
```
kubernetes.io/ingress.global-static-ip-name: static-ip-name 
```

######- Select which "StorageClass" the "PersistentVolumeClaim" will use:
```
volume.beta.kubernetes.io/storage-class: "generic" 
```

######- Using the `Alpha` annotation on the "PersistentVolumeClaim" will ignore the "StorageClass" completely:
```
volume.alpha.kubernetes.io/storage-class: "generic"
```
