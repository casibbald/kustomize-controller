# Development

> **Note:** Please take a look at <https://fluxcd.io/contributing/flux/>
> to find out about how to contribute to Flux and how to interact with the
> Flux Development team.

## Installing required dependencies

There are a number of dependencies required to be able to run the controller and its test suite locally:

- [Install Go](https://golang.org/doc/install)
- [Install Kustomize](https://kubernetes-sigs.github.io/kustomize/installation/)
- [Install Docker](https://docs.docker.com/engine/install/)
- (Optional) [Install Kubebuilder](https://book.kubebuilder.io/quick-start.html#installation)

## How to run the test suite

Prerequisites:
* Go >= 1.24

You can run the test suite by simply doing

```sh
make test
```

## How to run the controller locally

Install the controller's CRDs on your test cluster:

```sh
make install
```

Note that `kustomize-controller` depends on [source-controller](https://github.com/fluxcd/source-controller) to acquire its artifacts. If `source-controller` is not running on your test cluster, you need to tell `kustomize-controller` where to find it. 

Port forward to source-controller artifacts server:

```sh
kubectl -n flux-system port-forward svc/source-controller 8080:80
```

Export the local address as `SOURCE_CONTROLLER_LOCALHOST`:

```sh
export SOURCE_CONTROLLER_LOCALHOST=localhost:8080
```

Alternatively, if your test cluster is already running `source-controller` and `kustomize-controller`, you need to scale down the in-cluster `kustomize-controller`:

```
kubectl -n flux-system scale deployment/kustomize-controller --replicas=0
```

Run the controller locally:

```sh
make run
```

## How to install the controller

### Building the container image

Set the name of the container image to be created from the source code. This will be used when building, pushing and referring to the image on YAML files:

```sh
export IMG=registry-path/kustomize-controller:latest
```

Build the container image, tagging it as `$(IMG)`:

```sh
make docker-build
```

Push the image into the repository:

```sh
make docker-push
```

**Note**: `make docker-build` will build an image for the `amd64` architecture.


### Deploying into a cluster

Deploy `kustomize-controller` into the cluster that is configured in the local kubeconfig file (i.e. `~/.kube/config`):

```sh
make deploy
```

Running the above will also deploy `source-controller` and its CRDs to the cluster.