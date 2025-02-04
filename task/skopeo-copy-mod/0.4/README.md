# Skopeo Copy (modified)  (c) Oliver Liebel 2025

This Task uses [Skopeo](https://github.com/containers/skopeo), a command line tool
for working with remote image registries, to copy container images from one
registry to another. It supports both "internal" and "external" source registries
via authentication tokens. Skopeo does not require a daemon to be running while
performing these operations.

## Install the Task

```bash
kubectl apply -f https://raw.githubusercontent.com/k8strainer/tekton-catalog/refs/heads/main/task/skopeo-copy-mod/0.4/skopeo-copy-mod.yaml
```

*(If you have a modified version of the Task, adjust the URL or apply your local YAML.)*

## Parameters

- **srcImageURL**: URL of the image to be copied to the destination registry (e.g. `quay.io/myrepo/myimage:latest`)
  (_default:_ `""`)
- **destImageURL**: URL of the image where the image from source should be copied to (e.g. `gcr.io/my-other-repo/myimage:latest`)
  (_default:_ `""`)
- **srcTLSverify**: Verify the TLS on the source registry endpoint  
  (_default:_ `"true"_)
- **destTLSverify**: Verify the TLS on the destination registry endpoint  
  (_default:_ `"true"_)
- **multiArch**: How to handle multi-architecture images (`system`, `all`, or `index-only`)  
  (_default:_ `"system"_)
- **srcMode**: Either `"internal"` or `"external"` to indicate the source registry type. This determines which credentials (if any) will be used.  
  (_default:_ `""`)

## Workspaces

- **images-url**: A [Workspace](https://github.com/tektoncd/pipeline/blob/main/docs/workspaces.md) that can store or reference images, or hold additional context/config. Depending on your pipeline design, you can leave this empty if copying is done purely via registry endpoints.

## Results

This Task does not produce specific Tekton `results`.

## Platforms

This Task can be run on `linux/amd64`, `linux/s390x`, `linux/ppc64le`, and `linux/arm64`.

## Usage

Below is an example TaskRun that uses the `skopeo-copy` Task to copy an image from
an external registry (`quay.io`) to another registry (`gcr.io`):

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  name: skopeo-copy-sample
spec:
  taskRef:
    name: skopeo-copy
  params:
    - name: srcImageURL
      value: quay.io/myrepo/myimage:latest
    - name: destImageURL
      value: gcr.io/my-other-repo/myimage:latest
    - name: srcMode
      value: "external"  # or "internal" if pulling from an internal registry
  workspaces:
    - name: images-url
      emptyDir: {}  # or provide a PVC, if needed
```

Before running this TaskRun, ensure you have created and configured the appropriate
Secrets (for example `gcr-access-token`, `internal-registry-access`) as referenced
in the Task’s environment variables.
