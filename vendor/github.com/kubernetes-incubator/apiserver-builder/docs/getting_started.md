# Getting started (v0.0-alpha.12)

This document covers building your first apiserver from scratch:

- Bootstrapping your go dependencies
- Initialize your project directory structure and go packages
- Create an API group, version, resource
- Build the apiserver command
- Write an automated test for the API resource

## Download the latest release

Make sure you downloaded and installed the latest release as described
[here](https://github.com/kubernetes-incubator/apiserver-builder/blob/master/docs/installing.md)

## Create your Go project

Create a Go project under GOPATH/src/

For example

> GOPATH/src/github.com/my-org/my-project

## Create a copyright header

Create a file called `boilerplate.go.txt`.  This file will contain the
copyright boilerplate appearing at the top of all generated files.

Under GOPATH/src/github.com/my-org/my-project:

- `boilerplate.go.txt`

e.g.

```go
/*
Copyright 2017 The Kubernetes Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
```

## Initialize your project

This will setup the initial file structure for your apiserver, including
vendored go libraries pinned to a set of known working versions.  Vendored
libraries are distributed with the apiserver-builder binary tar and copied
from the installation directory.

Files created under GOPATH/src/github.com/my-org/my-project:

- `pkg/apis/doc.go`
- `pkg/openapi/doc.go`
- `docs/...`
- `main.go`
- `vendor/...`
- `glide.yaml`
- `glide.lock`


Flags:

- your-domain: unique namespace for your API groups

At the root of your go package under your GOPATH run the following command:

```sh
apiserver-boot init --domain <your-domain>
```

**Note:** You can skip vendoring by running with `--install-deps=false`, and install
the vendored deps separately with `apiserver-boot glide-install`.

## Create an API resource

An API resource provides REST endpoints for CRUD operations on a resource
type.  This is what will be used by clients to read and store instances
of the resource kind.

API resources are defined by a group (like a package), a version (v1alpha1, v1beta1, v1), and a Kind (the type)
Running the `apiserver-boot create-resource` command will create the api group, version and Kind for you.

Files created under GOPATH/src/github.com/my-org/my-project:

- `pkg/apis/your-group/doc.go`
- `pkg/apis/your-group/your-version/doc.go`
- `pkg/apis/your-group/your-version/your-kind_types.go`
- `pkg/apis/your-group/your-version/your-kind_types_test.go`

Flags:

- your-domain: unique namespace for your API groups.  must be the same as used with `init`.
- your-group: name of the API group e.g. `batch`
- your-version: name of the API version e.g. `v1beta1` or `v1`
- your-kind: **Upper CamelCase** name of the type e.g. `MyKind`

At the root of your go package under your GOPATH run the following command:

```sh
apiserver-boot create-resource --domain <your-domain> --group <your-group> --version <your-version> --kind <your-kind>
```

**Note:** The resource name is the lowercase pluralization of the kind e.g. `mykinds` and
generated by default.  To directly control the name of the resource, use the `--resource` flag.

**Note:** If desired, the api group and version maybe created as separate steps with
`apiserver-boot create-group` and `apiserver-boot create-version`.

## Generate the code and build the apiserver + controller-manager

Build the apiserver binary

```sh
apiserver-boot build
```

**Note:** This will automatically run the code generators.  These maybe
run separate using `apiserver-boot generate`.  It is possible to skip running
the generators as part of the build by using the flag `--generate=false`.
Note that the generators must be rerun any time new fields are added to your resources

## Run the apiserver + controller-manager

Run an etcd instance and the apiserver + controller-manager.

**Note:** must have etcd on your PATH

```sh
apiserver-boot run
```

Test with kubectl

```sh
kubectl --kubeconfig kubeconfig version
```

## Run a test

A placeholder test was created for your resource.  The test will
start your apiserver in memory, and allow you to create, read, and write
your resource types.

This is a good way to test validation and defaulting of your types.

```sh
go test pkg/apis/your-group/your-version/your-kind_types_test.go
```