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
Getting started with Juju
-------------------------

Juju handles provisioning machines and deploying complex systems to a
wide number of clouds, supporting service orchestration once the bundle of
services has been deployed.

**Table of Contents**

- [Prerequisites](#prerequisites)
   - [On Ubuntu](#on-ubuntu)
   - [With Docker](#with-docker)
- [Launch Kubernetes cluster](#launch-kubernetes-cluster)
- [Exploring the cluster](#exploring-the-cluster)
- [Run some containers!](#run-some-containers)
- [Scale out cluster](#scale-out-cluster)
- [Launch the "k8petstore" example app](#launch-the-k8petstore-example-app)
- [Tear down cluster](#tear-down-cluster)
- [More Info](#more-info)
    - [Cloud compatibility](#cloud-compatibility)


## Prerequisites

> Note: If you're running kube-up, on ubuntu - all of the dependencies
> will be handled for you. You may safely skip to the section:
> [Launch Kubernetes Cluster](#launch-kubernetes-cluster)

### On Ubuntu

[Install the Juju client](https://juju.ubuntu.com/install) on your
local ubuntu system:

    sudo add-apt-repository ppa:juju/stable
    sudo apt-get update
    sudo apt-get install juju-core juju-quickstart


### With Docker

If you are not using ubuntu or prefer the isolation of docker, you may
run the following:

    mkdir ~/.juju
    sudo docker run -v ~/.juju:/home/ubuntu/.juju -ti whitmo/jujubox:latest

At this point from either path you will have access to the `juju
quickstart` command.

To set up the credentials for your chosen cloud run:

    juju quickstart --constraints="mem=3.75G" -i

Follow the dialogue and choose `save` and `use`.  Quickstart will now
bootstrap the juju root node and setup the juju web based user
interface.


## Launch Kubernetes cluster

You will need to have the Kubernetes tools compiled before launching the cluster

    make all WHAT=cmd/kubectl
    export KUBERNETES_PROVIDER=juju
    cluster/kube-up.sh

If this is your first time running the `kube-up.sh` script, it will install
the required predependencies to get started with Juju, additionally it will
launch a curses based configuration utility allowing you to select your cloud
provider and enter the proper access credentials.

Next it will deploy the kubernetes master, etcd, 2 nodes with flannel based
Software Defined Networking.


## Exploring the cluster

Juju status provides information about each unit in the cluster:

    juju status --format=oneline
    - docker/0: 52.4.92.78 (started)
      - flannel-docker/0: 52.4.92.78 (started)
      - kubernetes/0: 52.4.92.78 (started)
    - docker/1: 52.6.104.142 (started)
      - flannel-docker/1: 52.6.104.142 (started)
      - kubernetes/1: 52.6.104.142 (started)
    - etcd/0: 52.5.216.210 (started) 4001/tcp
    - juju-gui/0: 52.5.205.174 (started) 80/tcp, 443/tcp
    - kubernetes-master/0: 52.6.19.238 (started) 8080/tcp

You can use `juju ssh` to access any of the units:

    juju ssh kubernetes-master/0


## Run some containers!

`kubectl` is available on the kubernetes master node.  We'll ssh in to
launch some containers, but one could use kubectl locally setting
KUBERNETES_MASTER to point at the ip of `kubernetes-master/0`.

No pods will be available before starting a container:

    kubectl get pods
    NAME             READY     STATUS    RESTARTS   AGE

    kubectl get replicationcontrollers
    CONTROLLER  CONTAINER(S)  IMAGE(S)  SELECTOR  REPLICAS

We'll follow the aws-coreos example. Create a pod manifest: `pod.json`

```
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "hello",
    "labels": {
      "name": "hello",
      "environment": "testing"
    }
  },
  "spec": {
    "containers": [{
      "name": "hello",
      "image": "quay.io/kelseyhightower/hello",
      "ports": [{
        "containerPort": 80,
        "hostPort": 80
      }]
    }]
  }
}
```

Create the pod with kubectl:

    kubectl create -f pod.json


Get info on the pod:

    kubectl get pods


To test the hello app, we need to locate which node is hosting
the container. Better tooling for using juju to introspect container
is in the works but we can use `juju run` and `juju status` to find
our hello app.

Exit out of our ssh session and run:

    juju run --unit kubernetes/0 "docker ps -n=1"
    ...
    juju run --unit kubernetes/1 "docker ps -n=1"
    CONTAINER ID        IMAGE                                  COMMAND             CREATED             STATUS              PORTS               NAMES
    02beb61339d8        quay.io/kelseyhightower/hello:latest   /hello              About an hour ago   Up About an hour                        k8s_hello....


We see `kubernetes/1` has our container, we can open port 80:

    juju run --unit kubernetes/1 "open-port 80"
    juju expose kubernetes
    sudo apt-get install curl
    curl $(juju status --format=oneline kubernetes/1 | cut -d' ' -f3)

Finally delete the pod:

    juju ssh kubernetes-master/0
    kubectl delete pods hello


## Scale out cluster

We can add node units like so:

    juju add-unit docker # creates unit docker/2, kubernetes/2, docker-flannel/2

## Launch the "k8petstore" example app

The [k8petstore example](../../examples/k8petstore/) is available as a
[juju action](https://jujucharms.com/docs/devel/actions).

    juju action do kubernetes-master/0

Note: this example includes curl statements to exercise the app, which automatically generates "petstore" transactions written to redis, and allows you to visualize the throughput in your browser.

## Tear down cluster

    ./kube-down.sh

or

    juju destroy-environment --force `juju env`

## More Info

Kubernetes Bundle on Github

 - [Bundle Repository](https://github.com/whitmo/bundle-kubernetes)
   * [Kubernetes master charm](https://github.com/whitmo/charm-kubernetes-master)
   * [Kubernetes node charm](https://github.com/whitmo/charm-kubernetes)
 - [Bundle Documentation](http://whitmo.github.io/bundle-kubernetes)
 - [More about Juju](https://juju.ubuntu.com)


### Cloud compatibility

Juju runs natively against a variety of cloud providers and can be
made to work against many more using a generic manual provider.

Provider          | v0.15.0
--------------    | -------
AWS               | TBD
HPCloud           | TBD
OpenStack         | TBD
Joyent            | TBD
Azure             | TBD
Digital Ocean     | TBD
MAAS (bare metal) | TBD
GCE               | TBD


Provider          | v0.8.1
--------------    | -------
AWS               | [Pass](http://reports.vapour.ws/charm-test-details/charm-bundle-test-parent-136)
HPCloud           | [Pass](http://reports.vapour.ws/charm-test-details/charm-bundle-test-parent-136)
OpenStack         | [Pass](http://reports.vapour.ws/charm-test-details/charm-bundle-test-parent-136)
Joyent            | [Pass](http://reports.vapour.ws/charm-test-details/charm-bundle-test-parent-136)
Azure             | TBD
Digital Ocean     | TBD
MAAS (bare metal) | TBD
GCE               | TBD


<!-- BEGIN MUNGE: GENERATED_ANALYTICS -->
[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/docs/getting-started-guides/juju.md?pixel)]()
<!-- END MUNGE: GENERATED_ANALYTICS -->
