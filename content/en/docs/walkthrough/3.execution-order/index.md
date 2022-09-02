---
title: Execution Order
linktitle: Execution Order
description: Create a task DAG
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "walkthrough"
        weight: 13
weight: 13
sections_weight: 13
categories: [testing,fundamentals]
draft: false
toc: true
---



Frisbee allows you to define the  workflow as a directed-acyclic graph (DAG) by specifying the  dependencies of each action. When you do so, the Frisbee  scheduler ensures that your action is run only after the specified dependencies have successfully completed. After they  succeed, the dependent action transitions from `INITIALIZING` to `PENDING` and then to `RUNNING`. If any of the job dependencies fail, the scenario automatically transitions from  `PENDING` to `FAILED`.



> For intended failures, such as services crashing as part of a Chaos experiment, see see `TolerateSpec` in the `Cluster` CRD. More info and example about this feature at [here](add link).



## DAG (Workflow Stages)

In the following workflow, step `A` runs first, as it has no dependencies. Once `A` has finished, steps `B` and `C` run in parallel. Finally, once `B` and `C` have completed, step `D` can run.

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: dag-diamond
spec:
  actions:
    - action: Service
      name: a
      service:
        templateRef: whalesay
        inputs:
          - { message: "I am A" }

    - action: Service
      name: b
      depends: { success: [ a ] }
      service:
        templateRef: whalesay
        inputs:
          - { message: "I am B" }

    - action: Service
      name: c
      depends: { success: [ a ] }
      service:
        templateRef: whalesay
        inputs:
          - { message: "I am C" }

    - action: Service
      name: d
      depends: { success: [ b, c ] }
      service:
        templateRef: whalesay
        inputs:
          - { message: "I am D" }
```

