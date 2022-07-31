---
title: Clustering
linktitle: Clustering
description: Run multiple Services in a shared context
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "walkthrough"
        weight: 15
weight: 15
sections_weight: 15
categories: [testing,fundamentals]
draft: false
toc: true
---



When writing testing scenarios, it is often very useful to be able to run multiple services in a shared execution context. 



## Create Clustered Services

There are two ways to create multiple services in a cluster: identical and parameterized.

#### Multiple Identical Services

The following snippet will create 4 identical services initialized with the given input (in the input is not defined, the default template values will be used).

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: clustering
spec:
  actions:
    # Create an iperf server
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    # Create a set of iperf clients
    - action: Cluster
      name: client
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 3 # Create multiple identical services
        inputs: 
          - { target: server }
```



As you may notice, another benefit of the `cluster` is that users can set dependencies on the entire cluster rather than individual services, thus greatly simplifying the dependency graph. 



#### Multiple Parameterized Services

The following snippet will create 3 parameterized services, with each service initialized with different `duration` values.

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: clustering
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
      cluster:
        templateRef: iperf.client
        inputs: 
        	# Iterate over the inputs to create parameterized services.
        	# Because the templates always expect `strings`, the duration 
        	# should be put in double quotes. Otherwise, you will a json error.
          - { target: server, duration: "10" }
          - { target: server, duration: "20" }
          - { target: server, duration: "30" }
```



#### Rules for mixing  `inputs` and `instances`.

| Instances  | Inputs      | Result                                                       |
| :--------- | :---------- | :----------------------------------------------------------- |
| N          | Undefined   | N `Instances` are created with the default tempalte values.  |
| Undefined  | N           | N `Instances` are created with custom `Inputs`.              |
| N (e.g 10) | <N (e.g 2)  | 10 `Instances` are created. Every 2 instances, the `Inputs` are repeated. |
| N (e.g 10) | <N (e.g 12) | Abort                                                        |







