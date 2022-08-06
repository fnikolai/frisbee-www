---
title: Simulate A Failure
linktitle: Simulate A Failure
description: 
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 26
weight: 26
sections_weight: 26
categories: [testing,fundamentals]
draft: false
toc: true
---



This document describes how to simulate  faults in Frisbee.



## Simulate Failures



```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: simulate-failure
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    # Create a cluster of clients
    - action: Cluster
      name: clients
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 3
        inputs:
          - { target: server }

    # Partition server from the clients; clients can reach the server, but server cannot reach the clients
    - action: Chaos
      name: partition
      depends: { running: [ clients ],  after: "30s" }
      chaos:
        templateRef: system.chaos.network.partition.partial
        inputs: # Handle the chaos template much like a service template.
          - { source: server, duration: 2m , direction: "to", dst: "clients-0, clients-1, clients-3" }
```





## Supported Failures

The next table presents an overview of the failure templates that exist in  `charts/system/templates/chaos`



​	

https://chaos-mesh.org/docs/simulate-network-chaos-on-kubernetes/
