| Go Report                                                                                                                                      | Travis                                                                                                             | CircleCI                                                                                                             | Azure Test                                                                                                                                                                                    | Azure Release                                                                                                                                                                                       | License                                                                                                                              |
|------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| [![Go Report Card](https://goreportcard.com/badge/github.com/oneinfra/oneinfra)](https://goreportcard.com/report/github.com/oneinfra/oneinfra) | [![Travis CI](https://travis-ci.org/oneinfra/oneinfra.svg?branch=master)](https://travis-ci.org/oneinfra/oneinfra) | [![CircleCI](https://circleci.com/gh/oneinfra/oneinfra.svg?style=shield)](https://circleci.com/gh/oneinfra/oneinfra) | [![Test Pipeline](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master)](https://dev.azure.com/oneinfra/oneinfra/_build/latest?definitionId=3&branchName=master) | [![Release Pipeline](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/release?branchName=master)](https://dev.azure.com/oneinfra/oneinfra/_build/latest?definitionId=4&branchName=master) | [![License: Apache 2.0](https://img.shields.io/badge/License-Apache2.0-brightgreen.svg)](https://opensource.org/licenses/Apache-2.0)|

# oneinfra

`oneinfra` is a Kubernetes as a Service platform. It empowers you to
provide or consume Kubernetes clusters at scale, on any platform or
service provider. You decide.

![Architecture diagram](docs/architecture.png)

You can [read more about its design here](docs/DESIGN.md).


## Managed Kubernetes versions

| Kubernetes version | Deployable with  | Default in       |                                                                                                                                                                            |                                                                                                                                                                             |
|--------------------|------------------|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `1.15.11`          | `20.04.0-alpha1` |                  | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.15.11)%20with%20local%20CRI%20endpoints)        | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.15.11)%20with%20remote%20CRI%20endpoints)        |
| `1.16.8`           | `20.04.0-alpha1` |                  | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.16.8)%20with%20local%20CRI%20endpoints)         | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.16.8)%20with%20remote%20CRI%20endpoints)         |
| `1.17.4`           | `20.04.0-alpha1` |                  | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.17.4)%20with%20local%20CRI%20endpoints)         | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.17.4)%20with%20remote%20CRI%20endpoints)         |
| `1.18.1`           | `20.04.0-alpha1` | `20.04.0-alpha1` | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.18.1)%20with%20local%20CRI%20endpoints)         | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.18.1)%20with%20remote%20CRI%20endpoints)         |
| `1.19.0-alpha.1`   | `20.04.0-alpha1` |                  | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.19.0-alpha.1)%20with%20local%20CRI%20endpoints) | ![Build Status](https://dev.azure.com/oneinfra/oneinfra/_apis/build/status/test?branchName=master&jobName=e2e%20tests%20(1.19.0-alpha.1)%20with%20remote%20CRI%20endpoints) |


## Go install

Build has been tested with go versions 1.13 and 1.14.

```console
$ GO111MODULE=on go get github.com/oneinfra/oneinfra/...@20.04.0-alpha1
```

This should have installed the following binaries:

* `oi-local-cluster`: allows you to test `oneinfra` locally in your
  machine, creating hypervisors as Docker containers.

* `oi`: CLI tool that allows you to test `oneinfra` locally in a
  standalone way, without requiring Kubernetes to store manifests.

* `oi-manager`: Kubernetes set of controllers that reconcile defined
  clusters.


## Quick start

For the quick start you can either leverage Kubernetes as a management
cluster, or you can go with the standalone approach if you don't want
to use Kubernetes.

* [Without Kubernetes (for testing purposes
  only)](#without-kubernetes-for-testing-purposes-only)
* [With Kubernetes as a management
  cluster](#with-kubernetes-as-a-management-cluster)


### Without Kubernetes (for testing purposes only)

* Requirements
  * Docker

If you don't want to deploy Kubernetes to test `oneinfra`, you can use
the `oi` CLI tool that will allow you to test the reconciliation
processes of `oneinfra` without the need of a Kubernetes cluster.

```console
$ oi-local-cluster cluster create | \
    oi cluster inject --name simple-cluster | \
    oi component inject --name controlplane1 --role control-plane | \
    oi component inject --name controlplane2 --role control-plane | \
    oi component inject --name controlplane3 --role control-plane | \
    oi component inject --name loadbalancer --role control-plane-ingress | \
    oi reconcile | \
    tee ha-cluster-manifests.conf | \
    oi cluster admin-kubeconfig > ha-cluster-kubeconfig.conf
```

And access it:

```console
$ kubectl --kubeconfig=ha-cluster-kubeconfig.conf cluster-info
Kubernetes master is running at https://172.17.0.4:30000
```

In this mode it's very important to understand that `oi` will read
manifests from `stdin` and output them into `stdout`, make sure you
keep a file up to date with the latest reconciled resources -- this is
why this model is not suitable for production.


### With Kubernetes as a management cluster

* Requirements
  * A Kubernetes cluster that will be the management cluster
  * The management cluster needs to reach the hypervisors you create
  * Docker, for creating fake local hypervisors

1. [Install
`kind`](https://github.com/kubernetes-sigs/kind#installation-and-usage). If
you already have a Kubernetes cluster you can use, you can skip this
step.

    ```console
    $ kind create cluster
    ```

2. Deploy `cert-manager` and `oneinfra`.

    ```console
    $ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.14.1/cert-manager.yaml
    $ kubectl wait --for=condition=Available deployment --timeout=2m -n cert-manager --all
    $ kubectl apply -f https://raw.githubusercontent.com/oneinfra/oneinfra/20.04.0-alpha1/config/generated/all.yaml
    $ kubectl wait --for=condition=Available deployment --timeout=2m -n oneinfra-system --all
    ```

3. Create a local set of fake hypervisors, so `oneinfra` can schedule
cluster control plane components somewhere. You can [also define your
own set of hypervisors](docs/hypervisors.md) if you prefer.

    ```console
    $ oi-local-cluster cluster create --remote | kubectl apply -f -
    ```

4. Now, create a managed cluster:

    ```console
    $ kubectl apply -f https://raw.githubusercontent.com/oneinfra/oneinfra/20.04.0-alpha1/config/samples/simple-cluster.yaml
    $ kubectl wait --for=condition=ReconcileSucceeded --timeout=2m cluster simple-cluster
    $ kubectl get cluster simple-cluster -o yaml | oi cluster admin-kubeconfig > simple-cluster-kubeconfig.conf
    ```

5. And access it:

    ```console
    $ kubectl --kubeconfig=simple-cluster-kubeconfig.conf cluster-info
    Kubernetes master is running at https://172.17.0.5:30000
    ```

6. (optional) You can then create a second managed cluster, this one
comprised by three control plane instances:

    ```console
    $ kubectl apply -f https://raw.githubusercontent.com/oneinfra/oneinfra/20.04.0-alpha1/config/samples/ha-cluster.yaml
    $ kubectl wait --for=condition=ReconcileSucceeded --timeout=2m cluster ha-cluster
    $ kubectl get cluster ha-cluster -o yaml | oi cluster admin-kubeconfig > ha-cluster-kubeconfig.conf
    ```

    1. And access it:

        ```console
        $ kubectl --kubeconfig=ha-cluster-kubeconfig.conf cluster-info
        Kubernetes master is running at https://172.17.0.5:30002
        ```


## Defining clusters

You can have a more detailed [read at the documentation on how to
define clusters](docs/clusters.md) once you have set up `oneinfra`.


## Joining worker nodes to a cluster

You can read more details about the [worker joining process
here](docs/joining-worker-nodes.md).


## License

`oneinfra` is licensed under the terms of the Apache 2.0 license.

```
Copyright (C) 2020 Rafael Fernández López <ereslibre@ereslibre.es>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
