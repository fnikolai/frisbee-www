---
title: Assertions
linktitle: Assertions
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "getting-started"
        weight: 19
weight: 19
sections_weight: 19
draft: true
toc: true
---



No testing framework can be complete without assertions. 

However, the scope of assertion is a tricky thing. 

Let's see how to use them properly.



## Assertions (The wrong way)



Let's try to repeat the `conditional` scenario we used before.

But, in this case, along with the `until` condition which `suspends` the scenario when the condition is met, we have added an `assert` to `abort` the scenario when the condition is met.



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

	# Scenario Scope
    - action: Cluster
      name: clients
      depends: { running: [ server ] }      
      assert: # Abort the scenario
          state: '{{.NumSuccessfulJobs}} >= 4'
	  # Cluster Scope          
      cluster: 			
        templateRef: iperf.client
        instances: 10 
        inputs:
          - { target: server, duration: "10" }
          - { target: server, duration: "20" }
          - { target: server, duration: "30" }
        schedule:
          cron: "@every 1m"
        until: # Suspend the cluster
          state: '{{.NumSuccessfulJobs}} >= 6'          
```



You wait, and wait, and wait, .... and the scenario keep creating clients. 

What happened ?


#### State expressions are locally scoped

As we explain previously for  `until`:,

* the expression `{{.NumSuccessfulJobs}} >= 6` is **local** to the `Cluster` and refers to the jobs managed by the `Cluster `controller. 

Equally for the `assertion`, 

* the expression`'{{.NumSuccessfulJobs}} >= 4'` is **local** to the `Scenario` and refers to the jobs managed by the `Scenario` controller. 

  

In other words, the `Scenario` accounts only for `Actions`, and not for jobs created within the context of a `Cluster`. The reason behind this convience will become clear later when we talk about `Cluster.Spec.Tolerate.FailedJobs`.



## Assertions (The right way)





Assertions (The wrong way)



