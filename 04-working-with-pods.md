# Demystifing Pods

## Pod augments workloads

```
# To show a complete list of Pod attributes
kubectl explain pods --recursive

# To view a specific Pod attributes and it's supported values
kubectl explain pod.spec.restartPolicy
```

## Pods enable resource sharing

- Pods run one or more containers.
- All the containers in the same Pod share the pod's execution env.
