# Cluster API Provider for Managed Bare Metal Hardware

This repository will contain a Machine actuator implementation for the
Kubernetes [Cluster API](https://github.com/kubernetes-sigs/cluster-api/).  It
is currently still under active development and is not yet functional.

For more information about this actuator and related repositories, see
http://metalkube.org/.

## Running the Actuator on OpenShift 4

Start with a running OpenShift 4 cluster.  At this stage, it doesn’t matter
what platform you start with.  These instructions were tested using a cluster
launched via https://github.com/code-ready/osp4/.

(Note that some of these instructions reflect the awkward state of this dev
environment.  This version of ‘code-ready’ had the machine-api-operator while
it was in the previous `openshift-cluster-api` namespace and used the upstream
machine API.  The fork of the machine-api-operator we’re trying to run expects
a different namespace, `openshift-machine-api`, and uses OpenShift’s fork of
the machine API.  The instructions will get cleaned up over time when redone
with a better matching starting environment.)

### Set machine-api-operator to unmanaged

We need a custom build of the machine-api-operator that knows how to run the
baremetal actuator.  This operator is managed by the Cluster Version Operator
(CVO), so we need to make sure the CVO will not undo our custom changes first.

One method is to specifically tell the CVO to stop managing the Machine API
Operator. There are some docs on this process here.  (If you work out the exact
steps, please submit them as a PR!)

https://github.com/openshift/cluster-version-operator/blob/master/docs/dev/clusterversion.md#setting-objects-unmanaged

A brute force approach is to just stop the CVO completely.

```
$ oc scale deployment cluster-version-operator -n openshift-cluster-version --replicas=0

$ oc get deployment cluster-version-operator -n openshift-cluster-version
NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
cluster-version-operator   0         0         0            0           6d
```

Then you can stop the Machine API Operator by scaling it down, as well.

```
$ oc scale deployment machine-api-operator -n openshift-cluster-api --replicas=0

$ oc get deployment machine-api-operator -n openshift-cluster-api
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
machine-api-operator   0         0         0            0           6d
```

Now we can run our custom Machine API Operator.

```
$ git clone https://github.com/russellb/machine-api-operator
$ cd machine-api-operator

$ oc create namespace openshift-machine-api
$ oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_02_machine.crd.yaml
$ oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_03_machine.crd.yaml
$ oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_03_machineset.crd.yaml
$ oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_04_machinedeployment.crd.yaml
$ oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_05_cluster.crd.yaml

$ make build
$ ./bin/machine-api-operator start --kubeconfig ~/Downloads/crc_libvirt_0.11.0/kubeconfig --images-json=pkg/operator/fixtures/images.json
```



... todo ...
