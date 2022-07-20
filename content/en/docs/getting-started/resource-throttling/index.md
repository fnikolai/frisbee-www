---
title: Resource Throttling
linktitle: Resource Throttling
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "getting-started"
        weight: 19
weight: 19
sections_weight: 19
draft: false
toc: true
---



In Kubernetes, when you create a specification for a pod, you can optionally specify how many resources each container is allowed to use on a [Kubernetes node](https://komodor.com/learn/kubernetes-nodes-complete-guide/). The most common resources are CPU and memory (RAM), but you can also specify others.



But, what happens when you have parameterized services? How can you give parameterized resources?



Frisbee address this issue via tha `decorators` abstraction.

* **Decorators** rewrite the pod's specification according to some rules, before submitting the pod for creation. This way, we can provide automation for adding telemetry agents and mounting volumes without exposing these concerns to the end-user.



## Parameterized Resources



```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: iperf.server
spec:
  inputs:       # Inputs allows for parameterized resources
    parameters:
      memory: ""
      cpu: ""
  service:
    decorators: # Decorators wrap Pod definitions with some nice features
      resources:
        cpu: { { "{{.Inputs.Parameters.cpu}}" } }
        memory: { { "{{.Inputs.Parameters.memory}}" } }
    containers:
      - name: app
        image: czero/iperf2
        command: [ iperf ]
        args: [ "-s", "-f", "m", "-i", "5" ]
```



## Resource Throttling



```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: conditionals
spec:
  actions:
    # Create an iperf server
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    # Create a set of iperf clients
    - action: Cluster
      name: clients
      depends: { running: [ server ] }
      assert:
        until:
          state: '{{.NumSuccessfulJobs}} >= 6'
      cluster:
        templateRef: iperf.client
        instances: 10 
        inputs: # Resources can be given as mere arguments
          - { target: server, duration: "10", cpu: "0.4", memory: "50Mi"  }
          - { target: server, duration: "20", cpu: "1", memory: "50i"  }
          - { target: server, duration: "30", cpu: "1.2", memory: "50i"  }
        schedule:
          cron: "@every 1m"
```



As can be seen, it is fairly easy to create numerous resources with random resources.
