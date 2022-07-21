# Deployment

A `Deployment`, is considered a type of `ReplicaSet` in Kubernetes.

It's a wrapper around a `podspec` which supports some scaling features. For example you can have a cluster of pods running the same container, and scale up or down
the number of instances running.

Imagine an API that you want to run on a few extra servers during peak hours, but scale down to fewer instances to save resources at night (or to free up resources to run data-intensive scheduled jobs overnight.)

You can also use this scaling property on something like Kafka consumers. Since Kafka consumers include features and protocols for sharing/splitting assignments of a Topic to a dynamic number of consumers, you can scale up the number of consumers reading from a Topic to make it read through the Topic faster, then scale it down when the consumers have caught up to the end of the Topic log.

Copy and paste this snippet out into its own file, and replace some of the strings with names for your deployment. Name the file something like `deployment.yml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: [appname]-deployment
  namespace: [namespace]
spec:
  replicas: 1
  selector:
    matchLabels:
      app: [appname]
  template:
    metadata:
      labels:
        app: [appname]
    spec:
      restartPolicy: Always
      imagePullSecrets:
        - name: overdrive-acr
      containers:
        - name: [appname]
          image: [image name]]
          args: ["dotnet", <dll name>]
          imagePullPolicy: IfNotPresent
          resources:
            limits:
              memory: 800Mi
              cpu: 800m
            requests:
              memory: 240Mi
              cpu: 160m
          env:
            - name: <name>
              value: <value>
            
            - name: <name2>                        
              valueFrom:
                secretKeyRef:
                  name: <name of secret>
                  key: <key-name within secret>

```

Now you can deploy your app, in a deployment, to kubernetes, with a single command

> kubectl apply -f ./deployment.yml

## summary

That did it.