###

- `Question`: What will happen if the value for the _limit_ that is set by _LimitRange_ may be less than the request value specified for the container in the spec that a client submits to the API server.

  ```
  apiVersion: v1
  kind: LimitRange
  metadata:
    name: cpu-resource-constraint
  spec:
    limits:
    - default: # this section defines default limits
        cpu: 500m
      defaultRequest: # this section defines default requests
        cpu: 500m
      max: # max and min define the limit range
        cpu: "1"
      min:
        cpu: 100m
      type: Container
  ```

  - And the Pod definition looks like this:

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: example-conflict-with-limitrange-cpu
  spec:
    containers:
    - name: demo
      image: registry.k8s.io/pause:3.8
      resources:
        requests:
          cpu: 700m
  ```

  - `Answer`: Pod will not be scheduled, failing with an error similar to this:

    ```
    Pod "example-conflict-with-limitrange-cpu" is invalid: spec.containers[0].resources.requests: Invalid value: "700m": must be less than or equal to cpu limit

    ```
