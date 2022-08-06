---
title: Distributed Logs
linktitle: Distributed Logs
description: Learn how to collect and view the logs from distributed services
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 29
weight: 29
sections_weight: 29
categories: [testing,fundamentals]
draft: false
toc: true
---



## Distributed Logs



#### Template

```
---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: iperf.client
spec:
  inputs:
    parameters:
      target: localhost
      logClaimName: ""
  service:
    decorators:
      telemetry:
        - system.telemetry.agent

    volumes: # Need to declare the volume here
      - name: logvolume
        persistentVolumeClaim:
          claimName: "{{.Inputs.Parameters.logClaimName}}"

    containers:
      - name: app
        image: czero/iperf2
        volumeMounts: # Need to mount the volume here
          - name: logvolume
            mountPath: /logs
        command:
          - /bin/sh
          - -c
          - |         
              set -eum
              cut -d ' ' -f 4 /proc/self/stat > /dev/shm/app
    
              iperf -c {{.Inputs.Parameters.target}} -t 5000 -i 5  >> /logs/${HOSTNAME}
```



Note that we use the `${HOSTNAME}` variable to dinstiguish the logs that come from different services.



#### Persistent Volume Claim

In order to have shared logs, we must first create a network volume.

This volume will then be mounted across the various containers.

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: logs
spec:
  storageClassName: network-volume
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
```



#### Scenario

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: distributed-logs
spec:
  actions:
    # Step 0. Enable the log viewer
    - action: Service
      name: logviewer
      service:
        templateRef: default.telemetry.logviewer
        inputs:
          - { logClaimName: logs }

    - action: Service
      depends: { running: [ logviewer ] }
      name: server
      service:
        templateRef: iperf.server

    - action: Cluster
      name: clients
      depends: { running: [ server ] }
      cluster:
        templateRef: iperf.client
        instances: 5
        inputs:
          - { target: server, logClaimName: logs  }
        schedule:
          cron: "@every 1m"
```




