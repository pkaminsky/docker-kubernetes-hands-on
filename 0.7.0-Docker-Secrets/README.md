# Secrets

Docker lets you store secrets. Docker secrets are viewable to people and pods in the same namespace, so that's about how secure they are.

If you had a shared cluster not locked down at all by something like RBACs, they wouldn't be secret at all, since all your users would see eachother's secrets. That would be fine if a single team used the cluster. Not so great if it's a shared cluster. Most K8s clusters are probably used by a lot of people to make them cost effective.

Secret files look something like this

```
apiVersion: v1
kind: Secret
metadata:
  name: <secret name>
  namespace: <namespace>
type: Opaque
stringData:
  key1: "value1"
  key2: "value2"
  etc3: "and-so-on"

```

Then a good way to use the secrets is to "mount" them as environment variables in the podspec.

```
[...]

          env:
            - name: <name>
              value: <value>
            - name: <name of environment variable the pod will see>
              valueFrom:
                secretKeyRef:
                  name: <secret name>
                  key: etc3 #the pod will see the value of "an-so-on" per the above

[...]
```