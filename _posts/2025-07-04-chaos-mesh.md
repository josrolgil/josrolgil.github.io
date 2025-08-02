---
layout: post
title: Using Chaos Mesh to generate I/O errors in Kubernetes
description: "A hands-on tutorial for testing storage resilience using Chaos Mesh to create I/O failures in Kubernetes clusters.
tags: [chaos-mesh, kubernetes, io-errors, chaos-testing]
excerpt_separator: <!--preview-->
last_modified_at: 2025-07-04
---
Chaos Mesh is a tool that allows to simulate different types of failures in your Kubernetes cluster.
<!--preview-->

![chaos_mesh_logo]({{ site.baseurl }}/images/2025/chaos/chaos-mesh-preview.png){: loading="lazy"}

Some time ago I needed to test an I/O error when configuring a service used by my application. One colleague suggested me Chaos Mesh tool. I bring below a short example about how to use Chaos Mesh to generate I/O errors in Kubernetes.

First, a brief introduction, it is an open source project for chaos testing. It offers different types of failures, targeting a Kubernetes cluster. You can leverage its basic test types and add new ones, being able to run single experiments or orchestrate more powerful ones. More in the
[documentation](https://chaos-mesh.org/docs/).

Regarding my experiment, I started installing Chaos Mesh in an already provisioned Kubernetes cluster with my application:

```
curl -sSL https://mirrors.chaos-mesh.org/v2.7.1/install.sh | bash -s -- -r containerd
```

In my case, the underlying container technology is containerd, therefore one can notice the '-r containerd' option. To install with Docker, you can just issue:

```
curl -sSL https://mirrors.chaos-mesh.org/v2.7.1/install.sh | bash -s -- -h
```

A namespaces chamos-mesh will be created. The different components are deployed there. Check the logs of the controller manager pod when troubleshooting the deployed chaos experiments is needed, it is the only hint you could have. Check also the CRDs created, it lists different types of tests.
Once the engine of Chaos Mesh is ready, you can apply the following I/O experiment:

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: IOChaos
metadata:
  name: disk-error
  namespace: chaos-mesh
spec:
  action: fault
  mode: one
  selector:
    namespaces:
      - test
    labelSelectors:
      app: test-app
  volumePath: /var/stats
  path: '/var/stats/**/*'
  containerNames:
     - test-app
  errno: 5 # IO error
  percent: 100
  duration: '400s'
```

This yaml is defining an errno 5 for IO error, targeting a namespace and label selector for my pod and the container I want to affect. To run the experiment:

```
kubectl -n chaos-mesh apply -f experiment.yaml
```

In my experience, I had some errors with the volume and path set, since my container had no write permissions. I checked the controller manager to see the error message. Also, I think, but I am not sure, that the running experiment affects to running pods but not now ones, I had to remove the experiment
and apply again when restarting the container, for instance, although I have to check this.
To verify the error, I issued:

```
kubectl -n test exec test-app-pod -c test-app ls -la mkdir /var/stats/newdir
```

and you will see the error message.

I did not spend more time learning about the tool because I accomplished my objective, but it seemed an easy tool to use when infrastructure errors want to be simulated and test in our applications.
