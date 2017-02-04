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

```javascript
$ cat FileWithSecret.json | base64 > secretFile
$ kubectl create secret generic secret-json --from-file=secretFile

$ kubectl describe secret secret-json

Name:           secret-json
Namespace:      default
Labels:         <none>
Annotations:    <none>
Type:   Opaque
Data
====
secretFile:  3153 bytes

```
