<!-- BEGIN MUNGE: UNVERSIONED_WARNING -->

<!-- BEGIN STRIP_FOR_RELEASE -->

![WARNING](http://kubernetes.io/img/warning.png)
![WARNING](http://kubernetes.io/img/warning.png)
![WARNING](http://kubernetes.io/img/warning.png)

<h1>PLEASE NOTE: This document applies to the HEAD of the source
tree only. If you are using a released version of Kubernetes, you almost
certainly want the docs that go with that version.</h1>

<strong>Documentation for specific releases can be found at
[releases.k8s.io](http://releases.k8s.io).</strong>

![WARNING](http://kubernetes.io/img/warning.png)
![WARNING](http://kubernetes.io/img/warning.png)
![WARNING](http://kubernetes.io/img/warning.png)

<!-- END STRIP_FOR_RELEASE -->

<!-- END MUNGE: UNVERSIONED_WARNING -->
# Downward API example

Following this example, you will create a pod with a containers that consumes the pod's name and
namespace using the [downward API](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/downward_api.md).

## Step Zero: Prerequisites

This example assumes you have a Kubernetes cluster installed and running, and that you have
installed the ```kubectl``` command line tool somewhere in your path. Please see the [getting
started](../../../docs/getting-started-guides/) for installation instructions for your platform.

## Step One: Create the pod

Containers consume the downward API using environment variables.  The downward API allows
containers to be injected with the name and namespace of the pod the container is in.

Use the [`examples/downward-api/dapi-pod.yaml`](dapi-pod.yaml) file to create a Pod with a container that consumes the
downward API.

```shell
$ kubectl create -f docs/user-guide/downward-api/dapi-pod.yaml
```

### Examine the logs

This pod runs the `env` command in a container that consumes the downward API.  You can grep
through the pod logs to see that the pod was injected with the correct values:

```shell
$ kubectl logs dapi-test-pod | grep POD_
2015-04-30T20:22:18.568024817Z POD_NAME=dapi-test-pod
2015-04-30T20:22:18.568087688Z POD_NAMESPACE=default
```


<!-- BEGIN MUNGE: GENERATED_ANALYTICS -->
[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/docs/user-guide/downward-api/README.md?pixel)]()
<!-- END MUNGE: GENERATED_ANALYTICS -->
