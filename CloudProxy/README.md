## Setting up `cloudproxy` for Kubernetes using Socket.

This example will show how to use `cloudproxy` in Kubernetes using Socket, for this you will need to share a volume between the `cloudproxy` container and the container that will connect to `MySQL`. For `PostgreSQL` the procedure is pretty much the same.

Along this page I will breakdown below yaml file in details:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: cloudproxy-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cloudproxy
  template:
    metadata:
      labels:
        app: cloudproxy
    spec:
      containers:
        - image: nginx
          name: nginx-server
          ports:
            - containerPort: 80
          volumeMounts:
          - name: secret-volume
            mountPath: /secret/
          - name: socket-connection
            mountPath: /cloudsql
        - image: gcr.io/cloudsql-docker/gce-proxy:1.09
          name: proxy-container
          volumeMounts:
          - name: socket-connection
            mountPath: /cloudsql
          - name: secret-volume
            mountPath: /secret/
          command: ["/cloud_sql_proxy", "-dir=/cloudsql", "-credential_file=/secret/key.json", "-instances=gke-cluster:us-central1:cloudtest"]
      volumes:
      - name: secret-volume
        secret:
          secretName: cloudproxy-creds
      - name: socket-connection
        emptyDir: {}
```
You can see above that there are 2 containers declared in the above YAML, the `nginx-server` and `proxy-container`.

- The `proxy-container` will create the socket connections to the database.

- The `nginx-server` will use the socket to connect to the database.

