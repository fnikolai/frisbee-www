---
title: Deletions
linktitle: Deletions
description: Resemble user driven deletions
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 24
weight: 24
sections_weight: 24
categories: [testing,fundamentals]
draft: false
toc: true
---



Running systems are far for being `static'. Nodes come-up, nodes get crashing, new clients arrive, old clients go away, etc etc.

Therefore, we should be able to test our systems against high-level of churns. 

And to do so, it is necessary not only to create new servers/clients, but also to be able to delete existing ones. 



## Deletion


```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: delete-job
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    # Create a Client
    - action: Service
      name: client
      depends: { running: [ server ] }
      service:
        templateRef: iperf.client
        inputs:
          - { target: server }

    # Delete the client, after 2 minutes
    - action: Delete
      name: delete-action
      depends: { after: "2m" }
      delete: # references jobs should be action.
        jobs: [ client ]
```



> **Referenced Jobs** are references to other actions that should be deleted -- you cannot delete `Services` within a `Cluster`. you can only delete the `Cluster`.





To verify the operation:

```shell
>> kubectl describe scenario
...
Events:
  Type    Reason        Age    From       Message
  ----    ------        ----   ----       -------
  Normal  Initialized   2m44s  deletions  Start scheduling jobs
  Normal  VExecBegin    44s    deletions  delete-action
  Normal  VExecBegin    44s    deletions  clients
  Normal  VExecSuccess  44s    deletions  clients
  Normal  VExecSuccess  44s    deletions  delete-action
```





