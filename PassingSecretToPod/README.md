#To create the secret object each item must be base64 encoded:

# echo -n "adminusername" | base64

YWRtaW51c2VybmFtZQ==

echo -n "myverysecretpassword" | base64

bXl2ZXJ5c2VjcmV0cGFzc3dvcmQ=

#The Object should be as follows:

apiVersion: v1
kind: Secret
metadata:
  name: thesecret
type: Opaque
data:
  password: bXl2ZXJ5c2VjcmV0cGFzc3dvcmQ=
  username: YWRtaW51c2VybmFtZQ==
