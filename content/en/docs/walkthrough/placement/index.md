---
title: Placement
linktitle: Placement
description: Control the service placement policy
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 20
weight: 20
sections_weight: 20
categories: [testing,fundamentals]
draft: false
toc: true
---


Kubernetes already provide a powerful mechanism control  which [node the Pod deploys to](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/). For example, to ensure that a Pod ends up on a node  with an SSD attached to it, or to co-locate Pods from two different services that communicate a lot into the same availability zone.


However, configuring these mechanisms is cumbersome. Especially when we talk about tens of dynamically created services. For this reason, Frisbee provides simple abstraction for controlling the placement at the level of a cluster.


## Placement


```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: placement
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    - action: Cluster
      name: group-a
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 10
        inputs:
          - { target: server, duration: "10" }
          - { target: server, duration: "20" }
          - { target: server, duration: "30" }
        # Place all clients on the same node.
        # Ensure that the node will be different
        # from where the server is running
        placement:
          collocate: true
          conflictsWith: [ server ]


    - action: Cluster
      name: group-b
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 10
        inputs:
          - { target: server, duration: "10" }
          - { target: server, duration: "20" }
          - { target: server, duration: "30" }
        # Place all clients on the same node.
        # Ensure that the node will be different
        # from where the server is running
        placement:
          collocate: false
          conflictsWith: [ server, group-a ]
```


The available fields are:


| Field         | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| collocate     | Place all the Services of this Cluster within the same node. |
| conflictsWith | Points to a list of Services, or a list of Clusters (whose services), cannot be located with this one. |
| Nodes         | Place all the Services of this Cluster within the specific set of nodes. |


> **WARNING**: If you run this scenarion on a local deployment, it will block forever since the constraints are not met.
