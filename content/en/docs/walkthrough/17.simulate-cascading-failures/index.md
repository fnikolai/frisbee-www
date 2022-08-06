---
title: Simulate Cascading Failures
linktitle: Simulate Cascading Failures
description: 
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 27
weight: 27
sections_weight: 27
categories: [testing,fundamentals]
draft: false
toc: true
---



This document describes how to simulate a cascade of  faults in Frisbee.



## Simulate Cascading Failures



```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: chaos-cascading-failures
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
        instances: 4
        inputs:
          - { target: server }

    # When all clients are up and running, kill some of them periodically
    - action: Cascade
      name: killer
      depends: { running: [ clients ] }
      cascade:
        templateRef: system.chaos.pod.kill
        inputs:
          - { target: clients-2 }
          - { target: clients-1 }
          - { target: clients-3 }
        schedule:
          cron: "@every 1m"
```



As you can see, the syntax of a  `Cascade` is similar to that of a `Cluster`, but instead of creating service jobs, you create fault-injection jobs. Like in a cluster, you can have your fault-injection jobs `time-driven`, `events-driven`, `conditional`, or `metrics-driven`.
