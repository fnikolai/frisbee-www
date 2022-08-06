---
title: Event-Driven Scheduling
linktitle: Event-Driven Scheduling
description: Learn how to drive the creation of services based on events
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



Users may also set the creation policy to construct variable workloads and dynamically changing topologies for elastic experiments. 



## Event-Driven Scheduling 

The next snippet shows how to schedule the creation of new services,  using  [`govaluate`](https://github.com/Knetic/govaluate) [syntax](https://github.com/Knetic/govaluate/blob/master/MANUAL.md).



```yaml
---
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
          cron: "@every 1m"   # Schedule one client every minute
          event:              # When 3 clients are created, give it a boost
            state: '{{.NumSuccessfulJobs}} >= 3'
```



#### Aggregation Functions

Notice that the event-driven mechanism is based on the following `aggregation functions`.

| Function                      | Description                                       |
| :---------------------------- | ------------------------------------------------- |
| IsPending(job string) bool    | returns true if the given job is Pending phase    |
| IsRunning(job string) bool    | returns true if the given job is Running phase    |
| IsSuccessful(job string) bool | returns true if the given job is Successful phase |
| IsFailed(job string) bool     | returns true if the given job is Failed phase     |
| NumPendingJobs() int          | returns the number of jobs in Pending Phase       |
| NumRunningJobs() int          | returns the number of jobs in Running Phase       |
| NumSuccessfulJobs() int       | returns the number of jobs in Successful Phase    |
| NumFailedJobs() int           | returns the number of jobs in Failed Phase        |
| ListPendingJobs() []string    | returns the name of jobs in Pending Phase         |
| ListRunningJobs() []string    | returns the name of jobs in Running Phase         |
| ListSuccessfulJobs() []string | returns the name of jobs in Successful Phase      |
| ListFailedJobs() []string     | returns the name of jobs in Failed Phase          |





#### Function Scope



The expression `{{.NumSuccessfulJobs}} >= 6` is **local** to the `Cluster` and refers to the jobs managed by the `Cluster` controller. 



In general, the scope of a Frisbee expression is local to the calling object. 

```bash
    - action: Cluster
   	  ... global scope ..
      cluster:
      ... local scope ..
```
