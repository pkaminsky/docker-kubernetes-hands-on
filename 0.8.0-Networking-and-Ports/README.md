# Networking and ports

There's different kinds of ports, but you have to expose them.

You have to expose them on the Docker image in the Dockerfile.

You also expose it on the podspec, minimal snippet here:

```
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
        - ports:
            - containerPort: 80

```

## Service and Ingress

To get things to connect, you need an ingress and a service.

Here's a single file example defining an ingress and service for an app named Catamaran. It would map to port 80 in the Deployment defined above.

```
apiVersion: v1
kind: Service
metadata:
  name: catamaran-service
spec:
  type: ClusterIP
  selector:
    app: catamaran
  ports:
  - name: catamaran-http
    port: 80
    targetPort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: catamaran
spec:
  tls:
    - hosts:
      - '#{Application.Url}'
      secretName: '#{TlsCert.SecretName}' 
  rules:
    - host: '#{Application.Url}'
      http:
        paths:
          - path: /
            backend:
              servicePort: catamaran-http
              serviceName: catamaran-service

```

But there's more of this that has to map up to the ingress controller on the cluster which is controlled by the admins.

Also DNS is an issue often outside an app developer's purview. You may only be able to reach the port via the node's IP address, which you can figure out from kubectl or equivalent tool.