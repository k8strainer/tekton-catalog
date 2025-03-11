# Modify Makefile Task

This Task searches for a specific Makefile in your Ansible Operator project, then
edits certain lines (e.g., `OPERATOR_SDK_VERSION`, `IMAGE_TAG_BASE`, and `IMG`) to
match the provided parameter values. It is useful for automatically updating or
customizing your Makefile before building and pushing your Operator.

## Install the Task

```bash
kubectl apply -f https://api.hub.tekton.dev/v1/resource/tekton/task/modify-makefile-task/0.1/raw
```

> If you have a modified version of the Task, apply your own YAML or local file
> path accordingly.

## Parameters

- **operator-sdk-version**:  
  The version of Operator SDK to inject into the Makefile.  
  *Example:* `v1.31.0`
  
- **image-tag-base**:  
  The base path or registry for the operator image tag.  
  *Example:* `quay.io/myrepo/`
  
- **version**:  
  The specific version or tag for the operator image.  
  *Example:* `v1.0.0`

## Workspaces

- **source**:  
  A [Workspace](https://github.com/tektoncd/pipeline/blob/main/docs/workspaces.md) containing the operator source code where the Makefile is located.

## Results

- **makefile-path**:  
  The absolute path to the identified (and modified) Makefile within the
  `source` workspace.

## Platforms

This Task can be run on:
- `linux/amd64`
- `linux/s390x`
- `linux/ppc64le`
- `linux/arm64`

## Usage

Below is an example `TaskRun` that uses the `modify-makefile-task` to locate and
update the Makefile for an Ansible Operator:

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  name: modify-makefile-example
spec:
  taskRef:
    name: modify-makefile-task
  params:
    - name: operator-sdk-version
      value: v1.31.0
    - name: image-tag-base
      value: quay.io/myrepo/
    - name: version
      value: v1.0.0
  workspaces:
    - name: source
      persistentVolumeClaim:
        claimName: operator-source-pvc
```

### Explanation

1. **Workspace**  
   The `source` workspace is bound to a PVC that includes an Ansible Operator
   project directory structure (e.g., `testdata/ansible/memcached-operator`).

2. **Parameters**  
   - `operator-sdk-version` sets the `OPERATOR_SDK_VERSION ?=` line in the Makefile.
   - `image-tag-base` replaces occurrences of `IMAGE_TAG_BASE ?=` and portions of
     `IMG ?=` or `BUNDLE_IMG ?=`.
   - `version` defines the image tag used for `IMG` and `BUNDLE_IMG` lines.

3. **Result**  
   The Task writes the absolute path of the located and modified Makefile to the
   `makefile-path` result, enabling downstream tasks (e.g., build tasks) to
   reference it easily.

By automating Makefile updates, this Task helps ensure that your Operator build
scripts always reflect the desired versions and image tags without manual editing.
