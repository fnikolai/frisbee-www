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
    weight: 23
weight: 23
sections_weight: 23
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
  name: deletions
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    # Create a first cluster
    - action: Cluster
      name: group-a
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 3

    # Create a first cluster
    - action: Cluster
      name: group-b
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 3

    # Delete the second cluster.
    - action: Delete
      name: delete-group-b
      depends: { running: [ group-b ],  after: "30s" }
      delete: # references jobs should be action.
        jobs: [ group-b ]
```



> **Referenced Jobs** are references to other actions that should be deleted -- you cannot delete `Services` within a `Cluster`. you can only delete the `Cluster`.

