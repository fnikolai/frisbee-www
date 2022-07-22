---
title: Naming and Addressing
linktitle: Naming and Addressing
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "getting-started"
        weight: 23
weight: 23
sections_weight: 23
draft: false
toc: true
---



In contrast to other Workflow enginers like Argo that use dynamically generated names, all objects in Frisbee follow a predictable naming pattern. This makes it possible to create communicating services.

## Reference an Object

In case of communicating services, such as a client-server architecture, we need the client to know the name of the server in other to establish a communication case. 

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: server-client
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf2.server

    - action: Service
      name: client
      depends: { running: [ server ] }
      service:
        templateRef: iperf2.client
        inputs: 
          - { server: "server"} # Direct Reference 
```



## Reference a Grouped Object

The `Cluster` is a Frisbee abstraction for grouping multiple services in a shared execution context. The services running within a cluster can be reference using the `sequential naming` pattern. For example, `servers-0, servers-1, ...servers-n`. 

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: server-client
spec:
  actions:
    - action: Cluster
      name: servers
      service:
        templateRef: iperf2.server
        instances: 2

    - action: Service
      name: client-a
      depends: { running: [ servers-0 ] }
      service:
        templateRef: iperf2.client
        inputs:
          - { server: "servers-0"} # Direct Reference 
          
	- action: Service
      name: client-b
      depends: { running: [ servers-1 ] }
      service:
        templateRef: iperf2.client
        inputs: 
          - { server: "servers-1"} # Direct Reference 
```



## Addressing Macros

The dynamic membership of Frisbee can cause numerous issues such as addressing a non-scheduled service or a removed one. To consistently address runtime objects, we provide addressing `macros` in the format `.{resource}.{name}.{filter}'`. 

The resource is the type of a Frisbee action, such as `service`, or `cluster.` The name is the name of the action. 



```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: server-client
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf2.server

    - action: Service
      name: client
      depends: { running: [ server ] }
      service:
        templateRef: iperf2.client
        inputs: # Indirect reference by macro
          - { server: .service.server.one }
```



When the workflow controller stumbles onto a macro, it queries the Kubernetes API for the desired object and replaces the macro with the returned results. In case there are multiple results, such as multiple services in a cluster, a subset of them can be selected via the following filters. 

| Filter  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| one     | select one object randomly                                   |
| all     | select all objects                                           |
| oldest  | select the oldest created object                             |
| latest  | select the most recently created object                      |
| percent | select a fixed % of the objects                              |
| first   | select the first of an ascending alphanumeric order of names |
| last    | select the last of an ascending alphanumeric order of names  |

