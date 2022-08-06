---
title: SLA Assertions
linktitle: SLA Assertions
description: Learn how to create SLA assertions
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 29
weight: 29
sections_weight: 29
categories: [testing,fundamentals]
draft: false
toc: true
---



## SLA Assertions

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: sla-assertions
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    - action: Service
      name: client
      depends: { running: [ server ] }
      assert:
        metrics: "avg() of query(summary/152/rx-max, 1m, now) is below(-3000000000)"
      service:
        templateRef: iperf.client
        inputs:
          - { target: server }

    # Create a cluster of noisy neighbors
    - action: Cluster
      name: noisy-neighbors
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 5
        inputs:
          - { target: server }
        schedule:
          cron: "@every 1m"
```





#### Metrics Syntax

Let's break down the expression`avg() of query(summary/152/rx-max, 1m, now) is below(-3000000000)`.

