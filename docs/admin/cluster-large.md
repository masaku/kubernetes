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
# Kubernetes Large Cluster

## Support
At v1.0, Kubernetes supports clusters up to 100 nodes with 30-50 pods per node and 1-2 container per pod (as defined in the [1.0 roadmap](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/roadmap.md#reliability-and-performance)).

## Setup

Normally the number of nodes in a cluster is controlled by the the value `NUM_MINIONS` in the platform-specific `config-default.sh` file (for example, see [GCE's `config-default.sh`](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/cluster/gce/config-default.sh)).

Simply changing that value to something very large, however, may cause the setup script to fail for many cloud providers. A GCE deployment, for example, will run in to quota issues and fail to bring the cluster up.

When setting up a large Kubernetes cluster, the following must be taken into consideration.

### Quota Issues

To avoid running into cloud provider quota issues, when creating a cluster with many nodes, consider:
* Increase the quota for things like CPU, IPs, etc.
  * In [GCE, for example,](https://cloud.google.com/compute/docs/resource-quotas) you'll want to increase the quota for:
    * CPUs
    * VM instances
    * Total persistent disk reserved
    * In-use IP addresses
    * Firewall Rules
    * Forwarding rules
    * Routes
    * Target pools
* Gating the setup script so that it brings up new node VMs in smaller batches with waits in between, because some cloud providers limit the number of VMs you can create during a given period.

### Addon Resources
To prevent memory leaks or other resource issues in [cluster addons](https://github.com/GoogleCloudPlatform/kubernetes/tree/master/cluster/addons/) from consuming all the resources available on a node, Kubernetes sets resource limits on addon containers to limit the CPU and Memory resources they can consume (See PR [#10653](https://github.com/GoogleCloudPlatform/kubernetes/pull/10653/files) and [#10778](https://github.com/GoogleCloudPlatform/kubernetes/pull/10778/files)).

For example:
```YAML
      containers:
        - image: gcr.io/google_containers/heapster:v0.15.0
          name: heapster
          resources:
            limits:
              cpu: 100m
              memory: 200Mi
```

These limits, however, are based on data collected from addons running on 4-node clusters (see [#10335](https://github.com/GoogleCloudPlatform/kubernetes/issues/10335#issuecomment-117861225)). The addons consume a lot more resources when running on large deployment clusters (see [#5880](https://github.com/GoogleCloudPlatform/kubernetes/issues/5880#issuecomment-113984085)). So, if a large cluster is deployed without adjusting these values, the addons may continuously get killed because they keep hitting the limits.

To avoid running into cluster addon resource issues, when creating a cluster with many nodes, consider the following:
* Scale memory and CPU limits for each of the following addons, if used, along with the size of cluster (there is one replica of each handling the entire cluster so memory and CPU usage tends to grow proportionally with size/load on cluster):
  * Heapster ([GCM/GCL backed](../../cluster/addons/cluster-monitoring/google/heapster-controller.yaml), [InfluxDB backed](../../cluster/addons/cluster-monitoring/influxdb/heapster-controller.yaml), [InfluxDB/GCL backed](../../cluster/addons/cluster-monitoring/googleinfluxdb/heapster-controller-combined.yaml), [standalone](../../cluster/addons/cluster-monitoring/standalone/heapster-controller.yaml))
  * [InfluxDB and Grafana](../../cluster/addons/cluster-monitoring/influxdb/influxdb-grafana-controller.yaml)
  * [skydns, kube2sky, and dns etcd](../../cluster/addons/dns/skydns-rc.yaml.in)
  * [Kibana](../../cluster/addons/fluentd-elasticsearch/kibana-controller.yaml)
* Scale number of replicas for the following addons, if used, along with the size of cluster (there are multiple replicas of each so increasing replicas should help handle increased load, but, since load per replica also increases slightly, also consider increasing CPU/memory limits):
  * [elasticsearch](../../cluster/addons/fluentd-elasticsearch/es-controller.yaml)
* Increase memory and CPU limits sligthly for each of the following addons, if used, along with the size of cluster (there is one replica per node but CPU/memory usage increases slightly along with cluster load/size as well):
  * [FluentD with ElasticSearch Plugin](../../cluster/saltbase/salt/fluentd-es/fluentd-es.yaml)
  * [FluentD with GCP Plugin](../../cluster/saltbase/salt/fluentd-gcp/fluentd-gcp.yaml)

For directions on how to detect if addon containers are hitting resource limits, see the [Troubleshooting section of Compute Resources](../user-guide/compute-resources.md#troubleshooting).


<!-- BEGIN MUNGE: GENERATED_ANALYTICS -->
[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/docs/admin/cluster-large.md?pixel)]()
<!-- END MUNGE: GENERATED_ANALYTICS -->
