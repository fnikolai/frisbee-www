---
title: Conditionals
linktitle: Conditionals
description: Build loops and conditional creation of events
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "walkthrough"
        weight: 17
weight: 17
sections_weight: 17
categories: [testing,fundamentals]
draft: false
toc: true
---



We also support conditional execution. The syntax is implemented by [`govaluate`](https://github.com/Knetic/govaluate) which offers the support for complex [syntax](https://github.com/Knetic/govaluate/blob/master/MANUAL.md).



## Conditionals



```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: conditionals
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    - action: Cluster
      name: clients
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 10 
        inputs:
          - { target: server, duration: "10" }
          - { target: server, duration: "20" }
          - { target: server, duration: "30" }
        schedule:
          cron: "@every 1m"
		# Suspend the execution if more than 6 clients are created
        until:
          state: '{{.NumSuccessfulJobs}} >= 6'
```



In this scenario, Frisbee will keep spawning clients, either it reaches 6 clients, after which it will be `suspended`. If suspended, the cluster will stop spawning new clients, but the old clients will remain active.



> NOTE: the expression `{{.NumSuccessfulJobs}} >= 6` is **local** to the `Cluster` and refers to the jobs managed by the `Cluster` controller. 

