# Bundle Build Task (c) Oliver Liebel 2025

This Task automates the process of building and pushing an Operator **Bundle** image
for an Ansible-based Operator. It uses a Makefile that defines the steps for
creating a bundle (via `make bundle`) and subsequently building and pushing it
(`make bundle-build bundle-push`).

## Install the Task

```bash
kubectl apply -f https://github.com/k8strainer/tekton-catalog/edit/main/task/bundle-build/0.1/raw
```

> If you have a custom or locally modified Task YAML, apply your own file instead.

## Parameters

- **image-registry**:  
  The registry URL where the Bundle image will be pushed.  
  *Example:* `quay.io/myrepo/`
  
- **image-tag**:  
  The specific tag to be used for the Bundle image.  
  *Example:* `my-operator-bundle:v1.0.0`
  
- **makefile-path**:  
  The absolute path to the `Makefile` within your `source` Workspace. This Task will
  navigate to the directory of the specified Makefile before running the build
  commands.  
  *Example:* `/workspace/source/operator-project/Makefile`

## Workspaces

- **source**:  
  A [Workspace](https://github.com/tektoncd/pipeline/blob/main/docs/workspaces.md) containing the source code for your Ansible Operator, including the Makefile that
  defines the Bundle build and push steps.

## Results

This Task does not produce specific Tekton `results`.

## Platforms

The Task can be run on:
- `linux/amd64`
- `linux/s390x`
- `linux/ppc64le`
- `linux/arm64`

## Usage

Below is an example `TaskRun` that demonstrates how to use the Bundle Build Task:

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  name: bundle-build-example
spec:
  taskRef:
    name: bundle-build-task
  params:
    - name: image-registry
      value: quay.io/myrepo/
    - name: image-tag
      value: my-operator-bundle:v1.0.0
    - name: makefile-path
      value: /workspace/source/build/Makefile
  workspaces:
    - name: source
      persistentVolumeClaim:
        claimName: my-operator-source
```

### Explanation

1. **Workspace**  
   The `source` workspace points to a PersistentVolumeClaim containing your
   Ansible Operator’s files, including the Makefile.

2. **Parameters**  
   - `image-registry` and `image-tag` combine to form the final Bundle image reference,
     e.g. `quay.io/myrepo/my-operator-bundle:v1.0.0`.
   - `makefile-path` tells the Task where your Makefile is located so it can run
     `make bundle` and `make bundle-build bundle-push`.

3. **Build & Push Steps**  
   In the Task, a privileged container changes to the Makefile directory and runs
   the `bundle` target first, followed by `bundle-build bundle-push`. These targets
   must be defined in your Makefile. This process packages your Operator as an
   Operator Bundle and publishes it to the specified registry.
