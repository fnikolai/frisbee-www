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



This document describes the basic features of Frisbee, including [Templates](#templates), [Actions](#actions), [Scenarios](#scenarios), [Visualized Operations](#visualized-operations), and [Security Guarantees](#security-guarantees).



## Templates

Templates define minimally constraining skeletons for Services and Faults.  When called without parameters, templates generate services initialized with reasonable defaults. With parameters, templates generate the customized configuration by replacing the placeholders (denoted as \{\{...\}\}) with the given input. Unlike other engines that evaluate templates at deployment time, Frisbee evaluates them at runtime. This strategy allows us to create a library of frequently-used specifications and use them to generate objects on-demand throughout the experiment.



## Actions

An Action  is the key to defining a DSL for replaying complex workloads. Frisbee covers a  full range of actions required to test a distributed system, and  consists of five comprehensive and fine-grained components: services, clusters, faults, and calls. 

* **Services:** The service is the standard representation of a containerized application that can be managed by the Frisbee controllers and combined in the testing scenario. 
* **Clusters:** The Cluster  is a Frisbee abstraction for managing multiple services in a shared execution context. 

* **Faults:**  Backed by [Chaos-Mesh](https://chaos-mesh.org/docs/basic-features/), Frisbee covers a full range of faults that might occur in a distributed system.

* **Calls:** Makes the Frisbee to run a command on a running services.

  



## Scenarios

A Frisbee scenario is a workflow of dependent actions that collectively describe the testing process, and an application  status check, so you can automate complete the entire process of a testing a system on Frisbee. The scenario  defines three important properties:

1. a list of actions that drive the testing process, 
2. the preconditions of the runtime before each action, and
3. the desired state of the runtime after each action.



Currently, Frisbee workflows provide the following features:

* **Logical Dependencies**: At every reconciliation loop, checks which conditions are met and triggers the appropriate action. If more than one condition is specified, AND operator is applied.

* **Event Sources**: 
  * Time-Driven: events are fired after an elapsed time measured by the Frisbee controller. 
  * State-Driven: events are fired by the Kubernetes API when managed objects have state changes, errors, or other messages that should be broadcast to the system.  
  * Metrics-Driven: events are fired by Grafana when the outcome of statistical analysis on the collected metrics matches a given rule, such as when percentile latency exceeds a given limit.
  * Tag-Driven: events are fired when one Frisbee controller passes contextual information to another Frisbee controller. 
* **Event-Driven Expressions**
  * Programmable Assertions
  * Conditional Loops
* **Addressing Macros**: The dynamicity of a Frisbee experiment can cause numerous issues such as addressing a non-scheduled service or a removed one. Macros are runtime evaluated expressions that forces the controller to query the Kubernetes API for the referenced object and replace the macro expression with the returned results. The results can be filter using on the following filters: first(), last(), oldest(), recent(), all(), one(), percent().
* **Phase and Conditions**: The phase of a Scenario is a simple, high-level summary of where the Scenario is in its lifecycle. The phase is not intended to be a comprehensive rollup of observations of scenario state, nor is it intended to be a comprehensive state machine. Instead, the Phase is a top-level description calculated based on some Conditions. The Conditions describe the various stages the Test has been through.





## Visualized Operations

Frisbee provides an extended suite for visualizing operations, which greatly simplifies the understanding of a system under testing. 





## Security Guarantees

Frisbee manages permissions using the native [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) feature in Kubernetes. You can freely create multiple roles based on your actual permission  requirements, bind the roles to the username service account, and then  generate the token corresponding to the service account. In addition, you can specify the namespaces that allow Frisbee experiments  by setting the namespace annotations, which further safeguards the  control of Frisbee experiments.
