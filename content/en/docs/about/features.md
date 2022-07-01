---
title: Basic Features
linktitle: Basic Features
description: 
date: 2022-06-30
publishdate: 2022-06-30
lastmod: 2022-06-30
menu:
  docs:
    parent: "about"
    weight: 20
weight: 20
sections_weight: 20
draft: false
toc: true
---



This document describes the basic features of Frisbee, including [Actions](#actions), [Testing Workflows](#testing-workflows), [Visualized Operations](#visualized-operations), and [Security Guarantees](#security-guarantees).



## Actions

An Action  is the key to defining a DSL for replaying complex workloads. Frisbee covers a  full range of actions required to test a distributed system, and  consists of five comprehensive and fine-grained components: templates, services, clusters, faults, and calls. 

* **Templates:** Templates define minimally constraining skeletons. When called with `input' parameters, templates generate the customized configuration by replacing the placeholders with the given input. 
* **Services:** The service is the standard representation of a containerized application that can be managed by the Frisbee controllers and combined in the testing scenario. 

* **Clusters:** The Cluster  is a Frisbee abstraction for managing multiple services in a shared execution context. 

* **Faults:**  Backed by [Chaos-Mesh](https://chaos-mesh.org/docs/basic-features/), Frisbee covers a full range of faults that might occur in a distributed system.

* **Calls:** Makes the Frisbee to run a command on a running services.

  



## Testing Workflows

The workflow comprise a collection of dependent actions that describe the testing process. It defines three important properties:

1. a list of actions that drive the testing process, 
2. the preconditions of the runtime before each action, and
3. the desired state of the runtime after each action.





* Wrap Action type into an Action.
* **Logical Dependencies**: At every reconciliation loop, checks which conditions are met and triggers the appropriate action. If more than one condition is specified, AND operator is applied.



* **Event Sources**: 
  * Time-Driven: events are fired after an elapsed time measured by the Frisbee controller. 
  * State-Driven: events are fired by the Kubernetes API when managed objects have state changes, errors, or other messages that should be broadcast to the system.  
  * Metrics-Driven: events are fired by Grafana when the outcome of statistical analysis on the collected metrics matches a given rule, such as when percentile latency exceeds a given limit.
  * Tag-Driven: events are fired when one Frisbee controller passes contextual information to another Frisbee controller. 

* **Event-Driven Expressions**
  * Assertions
  * Conditions

* **Addressing Macros**: The dynamicity of a Frisbee experiment can cause numerous issues such as addressing a non-scheduled service or a removed one. Macros are runtime evaluated expressions that forces the controller to query the Kubernetes API for the referenced object and replace the macro expression with the returned results. The results can be filter using on the following filters
  * first()
  * last()
  * oldest()
  * recent()
  * all()
  * one()
  * percent()

* **Phase and Conditions**: 





## Visualized Operations



When the workflow controller stumbles onto a macro, it queries Visualized operations

Frisbee provides an extended suite for visualizing operations, which greatly simplifies the understanding of a system under testing. 





## Security Guarantees



Frisbee manages permissions using the native [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) feature in Kubernetes.

You can freely create multiple roles based on your actual permission  requirements, bind the roles to the username service account, and then  generate the token corresponding to the service account. In addition, you can specify the namespaces that allow Frisbee experiments  by setting the namespace annotations, which further safeguards the  control of Frisbee experiments.
