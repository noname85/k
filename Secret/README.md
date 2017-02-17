####To create the secret object each item must be base64 encoded:

```javascript
$ echo -n "adminusername" | base64
YWRtaW51c2VybmFtZQ==

$ echo -n "myverysecretpassword" | base64
bXl2ZXJ5c2VjcmV0cGFzc3dvcmQ=
```
####Create a yaml file with below format
```javascript
apiVersion: v1
kind: Secret
metadata:
  name: thesecret
type: Opaque
data:
  password: bXl2ZXJ5c2VjcmV0cGFzc3dvcmQ=
  username: YWRtaW51c2VybmFtZQ==
```

####To create the secret from a file:
When creating a secret from file don't convert it to `base64` otherwise the information will be displayed in base64 on the POD`

```javascript
$ kubectl create secret generic secret-from-file --from-file=key.json

$ kubectl describe secret secret-from-file
Name:           secret-from-file
Namespace:      default
Labels:         <none>
Annotations:    <none>
Type:   Opaque
Data
====
key.json:       2323 bytes
```
####Using the Secret on a Deployment as environment variables:

``` 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: deployment-with-secret
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secrettest
  template:
    metadata:
      labels:
        app: secrettest
    spec:
      containers:
        - image: nginx
          name: nginx-server
          ports:
            - containerPort: 80
          env:
            - name: SECRET_USERNAME
              valueFrom:
                secretKeyRef:
                  name: thesecret
                  key: username
            - name: SECRET_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: thesecret
                  key: password       
```

####Using the Secret on a Deployment as a volume:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: deployment-with-secret
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secrettest
  template:
    metadata:
      labels:
        app: secrettest
    spec:
      containers:
        - image: nginx
          name: nginx-server
          ports:
            - containerPort: 80
          volumeMounts:
          - mountPath: "/mnt/secret"
            name: secret-volume
      volumes:
      - name: secret-volume
        secret:
          secretName: thesecret
```
