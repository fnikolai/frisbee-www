---
title: Conditional Scheduling
linktitle: Conditional Scheduling
description: Conditional Scheduling
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "walkthrough"
        weight: 18
weight: 18
sections_weight: 18
categories: [testing,fundamentals]
draft: false
toc: true
---



We also support loops execution. 



## Conditional Scheduling

In this scenario, Frisbee will keep spawning clients, either it reaches 6 clients, after which it will be `suspended`. If suspended, the cluster will stop spawning new clients, but the old clients will remain active.



```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: loops
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



