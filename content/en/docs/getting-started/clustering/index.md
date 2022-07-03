---
title: Clustering
linktitle: Clustering
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "getting-started"
        weight: 13
weight: 13
sections_weight: 13
draft: false
toc: true
---



When writing testing scenarios, it is often very useful to be able to run multiple services in a shared execution context. 



## Create Clustered Services

There are two ways to create services in a cluster: by instances and by inputs. 

#### By instances

```yaml
apiVersion: frisbee.io/v1alpha1
kind: TestPlan
metadata:
  name: whales-say
spec:
  actions:
    - action: Cluster	 
      name: whales-say
      cluster:
        templateRef: whalesay 
        instances: 4	# number of services
```

The above snippet will create 4 identical services initialized with the default values of the `whalesay` template. 

#### By inputs

```yaml
apiVersion: frisbee.io/v1alpha1
kind: TestPlan
metadata:
  name: whales-say
spec:
  actions:
    - action: Cluster	 
      name: whales-say
      cluster:
        templateRef: whalesay 
        # inputs specifies the list to iterate over
        inputs: 
          - { message: "I am A" } 
          - { message: "I am B" } 
          - { message: "I am C" } 
          - { message: "I am D" } 
```

The above snippet will create 4 parameterized services, with each service initialized with different values of the `whalesay` template. 

> When inputs are defined, the `instances` variable is automatically set to the number of specified inputs.



#### By custom inputs

There are cases in which we want to create identical services, but whose parameters are different from the template's default values. 

In this case, we use a single set of `inputs` to define the custom values, and then we specify the number of desired instances. 





```yaml
apiVersion: frisbee.io/v1alpha1
kind: TestPlan
metadata:
  name: whales-say
spec:
  actions:
    - action: Cluster	 
      name: whales-say
      cluster:
        templateRef: whalesay 
        instances: 4
        inputs: 
          - { message: "I am Groot" } 
```



> When `inputs` and `instances` are used in parallel, only one set of `inputs` can be defined. Otherwise, the scenario will abort immediately.



## Scheduling Services

Users may also set the creation policy to construct variable workloads and dynamically changing topologies for elastic experiments. The next snippet shows how to schedule the creation of new services, using a cron-like syntax.

```yaml
apiVersion: frisbee.io/v1alpha1
kind: TestPlan
metadata:
  name: whales-say
spec:
  actions:
    - action: Cluster	 
      name: whales-say
      cluster:
        templateRef: whalesay 
        inputs: 
          - { message: "I am A" } 
          - { message: "I am B" } 
          - { message: "I am C" } 
          - { message: "I am D" } 
        schedule: # Create the service periodically
      		cron: "@every 1m"
```





