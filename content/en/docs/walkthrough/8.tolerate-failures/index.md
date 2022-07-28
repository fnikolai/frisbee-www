---
title: Tolerate Failures
linktitle: Tolerate Failures
description: Learn how to perform recoverability experiments
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



Systems fail. After all, this is what testing is about. 



By default, Frisbee has  a builtin `fail-fast` mechanism that aborts the experiment when a fault is sensed.

## Tolerate Failures



#### Template

Instead of waiting for a `service` to fail, let's give it some boost.

We have modified the previous template to take the exit code as an input.

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: iperf.client
spec:
  inputs:
    parameters:
      target: localhost
      duration: "60"
      exit: "false" # Take exit code as an input
  service:
    containers:
      - name: app
        image: czero/iperf2
        command:
          - /bin/sh   # Run shell
          - -c        # Read from string
          - |         # Multi-line str
            # Compare the input, and exit if needed
            [[ {{.Inputs.Parameters.exit}} == "true" ]] && exit -1
            
            # Otherwise, continue as normal
            iperf -c {{.Inputs.Parameters.target}} -t {{.Inputs.Parameters.duration}} 
```



Note that instead of the default `command: [ iperf ]`, we are starting the container using a custom script.



#### Non failure-tolerant Scenario

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: tolerate-failures
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
          - { target: server, duration: "20", exit: "true" }
          - { target: server, duration: "30" }
        schedule:
          cron: "@every 1m"
```



Very roughly the execution sequence will be:

```tex
0-1m: sleep
1m: create clients-1
2m: create clients-2 (failed)
-- Frisbee aborts the experiment --
```



You can confirm that by inspecting the status of the scenario:

`kubectl get scenarios tolerate-failures  -o jsonpath='{.status.phase}'`

For more automation, use: 

`watch -d (date +TIME:%H:%M:%S && kubectl get scenarios tolerate-failures  -o jsonpath={.status.phase}) | tee /tmp/logs`



#### Failure-tolerant Scenario

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: tolerate-failures
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
          - { target: server, duration: "10"  }
          - { target: server, duration: "20", exit: "true" }
          - { target: server, duration: "30" }
        schedule:
          cron: "@every 1m"
        # Tolerate up to 2 failing clients, before the cluster fails itself
        tolerate: 
          failedJobs: 3
```



Very roughly the execution sequence will be:

```tex
0-1m: sleep
1m: create clients-1
2m: create clients-2 (failed)
3m: create clients-3
4m: create clients-4 (failed)
5m: create clients-5
6m: create clients-6 (failed)
-- Frisbee aborts the experiment --
```

