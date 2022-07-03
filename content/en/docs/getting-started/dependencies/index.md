---
title: Dependencies
linktitle: Dependencies
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



Frisbee allows you to define the  workflow as a directed-acyclic graph (DAG) by specifying the  dependencies of each task. This can be simpler to maintain for complex  workflows and allows for a greater range of test-cases. 



## Run Sequentially

In the following workflow, step `A` runs first, as it has no dependencies. Once `A` has finished, steps `B` and `C` run in parallel. Finally, once `B` and `C` have completed, step `D` can run.

```yaml
apiVersion: frisbee.io/v1alpha1
kind: TestPlan
metadata:
  name: dag-diamond
spec:
  actions:
    - action: Service
      name: A
      service:
        templateRef: whalesay 
        inputs:
          - { msg: "I am A" }        
  
    - action: Service
      name: B
      depends: { success: [ A ] }
      service:
        templateRef: whalesay
        inputs:
          - { msg: "I am B" }
          
    - action: Service
      name: C
      depends: { success: [ A ] }
      service:
        templateRef: whalesay
        inputs:
          - { msg: "I am C" }          

    - action: Service
      name: D
      depends: { success: [ B, C ] }
      service:
        templateRef: whalesay
        inputs:
          - { msg: "I am D" }   
```



The DAG logic has a built-in `fail fast` feature to abort the workflow as soon as it detects that one of the DAG nodes is failed. For intended failures, such as services crashing as part of a Chaos experiment, see see `TolerateSpec` in the `Cluster` CRD. More info and example about this feature at [here](add link).



## Run In Parallel

In the above workflow, one action starts after a previous action is complete. This kind of semantic is useful for batch jobs, but not for distributed computing. This is because services (and servers in general) are usually long-running that never complete.  

```yaml
apiVersion: frisbee.io/v1alpha1
kind: TestPlan
metadata:
  name: dag-diamond
spec:
  actions:
  	# A runs before the following steps.
    - action: Service 
      name: A
      service:
        templateRef: whalesay 
        inputs:
          - { msg: "I am B", delay: "10s" }        
  
  	# B runs in parallel with A.
    - action: Service
      name: B
      depends: { running: [ A ] }
      service:
        templateRef: whalesay
        inputs:
          - { msg: "I am B",  delay: "40s"  }
          
	# C runs after A is complete.
    - action: Service
      name: C
      depends: { success: [ A ] }
      service:
        templateRef: whalesay
        inputs:
          - { msg: "I am C", delay: "20s"  }         

	# D runs after C is complete, and while B is still running.
    - action: Service
      name: D
      depends: { running: [ B ], success: [ C] }
      service:
        templateRef: whalesay
        inputs:
          - { msg: "I am D" }   
```



In addition to the `success` semantics, Frisbee workflows also support `running` semantics. If used, a new action starts after a previous action is still running. As we will see [next](macros), this is an extremely useful features for handling dependencies in server-client architectures, where the client must start only after the server is up and running. 



> Combining `success` and `running` on the same dependencies set can be tricky. If, for example, B completes before C, then the evaluation of`{ running: [ B ], success: [ C] ] }` will fail since B is no longer running.





## Cluster Dependencies 

Dependencies may be among any Frisbee action. 



```yaml
apiVersion: frisbee.io/v1alpha1
kind: TestPlan
metadata:
  name: whales-say
spec:
  actions:
  
  
	- action: Cluster	 
      name: one-group
      cluster:
        templateRef: whalesay 
        inputs: 
          - { message: "I am A" } 
          - { message: "I am B" }  
  
	- action: Cluster	 
      name: another-group
      depends: { success: [one-group] }
      cluster:
        templateRef: whalesay 
        inputs: 
          - { message: "I am C" } 
          - { message: "I am D" }           
```

As you may notice, another benefit of the `cluster` is that users can set dependencies on the entire cluster rather than individual services, thus greatly simplifying the dependency graph. 

