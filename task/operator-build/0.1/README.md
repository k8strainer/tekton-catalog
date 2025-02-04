# Operator Build Task (c) Oliver Liebel 2025

This Task automates the process of building and pushing an Ansible-based Operator
image to a container registry. It expects a Makefile that defines the `docker-build`
and `docker-push` targets, allowing you to customize how your Operator is built
and published.

In typical usage, you reference this Task in your Pipeline. After checking out the
source code (including the Makefile), the Task will navigate to the directory of
the specified Makefile and run the corresponding commands to build and push your
Operator image.

## Install the Task

```bash
kubectl apply -f https://github.com/k8strainer/tekton-catalog/blob/main/task/operator-build/0.1/raw
```

> If you have a custom or locally modified Task, apply your own YAML instead.

## Parameters

- **image-registry**:  
  The registry URL to which the Operator image will be pushed.  
  *Example:* `quay.io/myrepo/`
  
- **image-tag**:  
  The tag that will be appended to the image name.  
  *Example:* `my-operator:v1.0.0`
  
- **makefile-path**:  
  The absolute path to the `Makefile` within the `source` Workspace. This Task will
  change into the directory containing the Makefile before running the build and
  push commands.  
  *Example:* `/workspace/source/build/Makefile`

## Workspaces

- **source**:  
  A [Workspace](https://github.com/tektoncd/pipeline/blob/main/docs/workspaces.md) containing the Ansible Operator’s source code and its Makefile.

## Results

This Task does not explicitly set any Tekton `results`.

## Platforms

The Task can be run on:
- `linux/amd64`
- `linux/s390x`
- `linux/ppc64le`
- `linux/arm64`

## Usage

Below is an example `TaskRun` illustrating how to use the Operator Build Task:

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  name: operator-build-example
spec:
  taskRef:
    name: operator-build-task
  params:
    - name: image-registry
      value: gcr.io/my-registry/
    - name: image-tag
      value: my-ansible-operator:v1.0.0
    - name: makefile-path
      value: /workspace/source/build/Makefile
  workspaces:
    - name: source
      persistentVolumeClaim:
        claimName: my-source
```

### Explanation

1. **Workspace**:  
   The `source` workspace is bound to a PersistentVolumeClaim, which contains
   your operator source code (including the Makefile).

2. **Params**:  
   - `image-registry` and `image-tag` together define the final image reference
     (e.g. `gcr.io/my-registry/my-ansible-operator:v1.0.0`).
   - `makefile-path` points the Task to the exact location of your Makefile.

3. **Build and Push**:  
   Inside the Task, a privileged container (for Build operations) changes into the
   Makefile directory and runs `make docker-build docker-push`, which must be
   defined in your Makefile. These targets are responsible for building the
   operator image and pushing it to the registry.

This approach provides a clean separation of responsibilities: the Task handles the
high-level build steps via Make, and your Makefile determines the details of how
the build and push are performed.
