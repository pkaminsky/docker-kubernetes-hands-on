## You can make a podspec

A `pod` is a wrapper around one or more containers.

> Technically a pod can run multiple containers, and there's reasons to do that, but a lot of the time it's simpler and cleaner to just do 1:1 containers and pods. I would argue that this is also thematically more appropriate with the idea behind containerization. Still, there's some reasons why you'd use another container within a pod.

Copy this snippet to its own file called something like `podspec.yml`, then customize some of the properties.


```
apiVersion: v1
kind: Pod
metadata:
  name: <appname>
  namespace: data-integration
  labels:
    app: <appname>
spec:
  restartPolicy: Never
  imagePullSecrets:
    - name: overdrive-acr
  containers:
    - name: <appname>
      image: <image>
      args: ["sh", "-c", "tail -f /dev/null"]
      imagePullPolicy: IfNotPresent
      resources:
        limits:
          memory: 400Mi
          cpu: "2.5"
        requests:
          memory: 250Mi
          cpu: "1"
```

Now you can deploy this to kubernetes.

> kubectl apply -f ./podspec.yml


## Summary

You can deploy a pod to kubernetes, but in practice we often wrap them in something else.