---
title: Dependency Order
linktitle: Dependency Order
description: Spin-up the dependency stack of a system
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "walkthrough"
        weight: 14
weight: 14
sections_weight: 14
categories: [testing,fundamentals]
draft: false
toc: true
---



In the `Execution Order` example, one action starts after a previous action is complete. This kind of semantic is useful for batch jobs, but not for distributed computing. This is because services (and servers in general) are usually long-running that never complete.  



In addition to the `success` semantics, Frisbee workflows also support `running` semantics. If used, a new action starts after a previous action is still running. 



## Iperf Templates

Similarly to what we did in the `hello-world` example, we must create the templates that will be used in the testing scenario.

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: iperf.server
spec:
  service:
    containers:
      - name: app
        image:  czero/iperf2
        ports:
          - name: listen
            containerPort: 5001        
        command: [iperf]
        args: ["-s", "-f", "m", "-i", "5"]


---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: iperf.client
spec:
  inputs:
    parameters:
      target: localhost
  service:
    containers:
      - name: app
        image:  czero/iperf2
        command: [iperf]
        args: ["-c", "{{.Inputs.Parameters.target}}" ]
```





## DAG (Client-Server)

As we see , `running` dependencies can be extremely helpful in server-client architectures, where the client must start only after the server is up and running. 

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: dependency-order
spec:
  actions:
    # Create an iperf server
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    # Create an iperf client
    - action: Service
      name: client
      depends: { running: [ server ] }
      service:
        templateRef: iperf.client
        inputs:
          - { target: server }
```



To see the logs use the `kubectl logs` command.

```yaml
kubectl logs server
```



> Combining `success` and `running` on the same dependencies set can be tricky. If, for example, B completes before C, then the evaluation of`{ running: [ B ], success: [ C] ] }` will fail since B is no longer running.



