---
title: Revoke Failures
linktitle: Revoke Failures
description: Learn how to revoke failures manually.
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 28
weight: 28
sections_weight: 28
categories: [testing,fundamentals]
draft: false
toc: true
---



Failures in Frisbee can be revoked in two ways:

* Automatically after an expiration time
* Manually



## Revoke Failures Automatically

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: revoke-failure
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    - action: Service
      name: client
      depends: { running: [ server ] }
      service:
        templateRef: iperf.client
        inputs:
          - { target: server }

    # Inject a network failure, after 1 minute, 
    # and let run for 2minutes.
    - action: Chaos
      name: fault
      depends: { running: [ client ], after: "1m" }
      chaos:
        templateRef: system.chaos.network.partition.partial
        inputs:
          - { source: server, duration: "2m", dst: client }
```



## Revoke Failures Manually

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: revoke-failure
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    - action: Service
      name: client
      depends: { running: [ server ] }
      service:
        templateRef: iperf.client
        inputs:
          - { target: server }

    # Inject a network failure, after 1 minute
    - action: Chaos
      name: fault
      depends: { running: [ client ], after: "1m" }
      chaos:
        templateRef: system.chaos.network.partition.partial
        inputs:
          - { source: server, dst: client }

    # Repair the partition (by deleting the chaos job),
    # after it has run for two minutes (3m-1m)
    - action: Delete
      name: delete-fault
      depends: { after: "3m" }
      delete:
        jobs: [ fault ]
```



