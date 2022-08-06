---
title: Callables
linktitle: Callables
description: Learn how to infer with a running system
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 22
weight: 22
sections_weight: 22
categories: [testing,fundamentals]
draft: false
toc: true
---


There are cases where you need to get a shell to a running container.

For this reason, Frisbee provides the `Call` primitive.


> **Hints**:
>
> 1. The number of expected outputs must be the same as the number of defined services.
> 2. Because the `Call` primitive is synchronous, it may block the scenario if the referenced services does not exist.
> 3. The expected output for and `stdout` and `stderr` is a regex. For "raw" matching, the expression cannot be more than 1024 characters.



## Remote Calls


#### Template

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: whalesay
spec:
  inputs:
    parameters:
      message: "hello-world"
  service:
    containers:
      - name: app
        image: docker/whalesay
        command: [ "tail", "-f", "/dev/null" ]

    # Alter the container's execution, at runtine
    callables:
      launch:
        container: app                                            # Target container
        command:  [ "cowsay", "{{.Inputs.Parameters.message}}" ]  # Function to call
```


#### Scenario

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name:  callables
spec:
  actions:
    # Provision a set of idle pods
    - action: Cluster
      name: idle
      cluster:
        templateRef: whalesay
        inputs:
          - { message: "I am A" }
          - { message: "I am B" }
          - { message: "I am C" }
          - { message: "I am D" }

    # Invoke callables into the idle pods
    - action: Call
      name: group-a
      depends: { running: [ idle ] }
      call:
        callable: launch                              # Function to call
        services: [idle-1, idle-2, idle-0, idle-1]    # List of target services
        expect:
          - stdout: "I am B"
          - stdout: "I am C"
          - stdout: "oops .."
          - stdout: "I am D"
```

In the Call, you can optionally define `regex` for matching the logs in the `stdour` and `stderr`.


If the output does not match the expected, the Call is `failed`.


To validate the execution:

```bash
>> kubectl describe call
...
Events:
    Type     Reason       Age   From       Message
    ----     ------       ----  ----       -------
    Normal   CallBegin    29s   group-a-0  idle-1
    Normal   CallSuccess  29s   group-a-0  idle-1
    Normal   CallBegin    29s   group-a-1  idle-2
    Normal   CallSuccess  29s   group-a-1  idle-2
    Normal   CallBegin    29s   group-a-2  idle-0
    Warning  CallFailed   29s   group-a-2  idle-0
```

To get the reason for the failure of the `Call`:

```
>> kubectl get calls group-a -o jsonpath='{.status.reason}'
JobHasFailed
```



## Failure-Tolerate Remote Calls

Calls may `fail` either due to bad connectivity, or to output mismatch.

Regardless the reason, you can tolerate an error by using the `tolerate` primitive (as in the `Cluster`).


```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name:  remote-calls
spec:
  actions:
    # Provision a set of idle pods
    - action: Cluster
      name: idle
      cluster:
        templateRef: whalesay
        inputs:
          - { message: "I am A" }
          - { message: "I am B" }
          - { message: "I am C" }
          - { message: "I am D" }

    # Invoke callables into the idle pods
    - action: Call
      name: group-a
      depends: { running: [ idle ] }
      call:
        callable: launch                              # Function to call
        services: [idle-1, idle-2, idle-0, idle-1]    # List of target services
        expect:
          - stdout: "I am B"
          - stdout: "I am C"
          - stdout: "oops .."
          - stdout: "I am D"
        tolerate:                                     # Continue despite an error
          failedJobs: 1
```


To validate the execution:

```bash
>> kubectl describe call
...
Events:
    Type     Reason       Age   From       Message
    ----     ------       ----  ----       -------
    Normal   CallBegin    11s   group-a-0  idle-1
    Normal   CallSuccess  11s   group-a-0  idle-1
    Normal   CallBegin    11s   group-a-1  idle-2
    Normal   CallSuccess  11s   group-a-1  idle-2
    Normal   CallBegin    11s   group-a-2  idle-0
    Warning  CallFailed   11s   group-a-2  idle-0
    Normal   CallBegin    11s   group-a-3  idle-1
    Warning  CallFailed   11s   group-a-3  idle-1
```

To get the reason for the failure:

```bash
>> kubectl get calls group-a -o jsonpath='{.status.reason}'
TooManyJobsHaveFailed
```



## Understanding the Call abstraction


The `Call` is a composite abstraction like a `Cluster`.

The `Cluster` consists of `Service` jobs, whereas the `Call` consists of `VirtualObject` jobs.


If the `Cluster` fails, we have to inspect the individual `Services` to understand the reason of the failure. Equally, for the `Call`, we have to inspect into the `VirtualObject`.


Let's try to understand why the previous failure-tolerate scenario has failed.

#### Step 1. Get a high-lever overview of the Call

```bash
>> kubectl get calls group-a -o jsonpath='{.status.phase}'
Failed

>> kubectl get calls group-a -o jsonpath='{.status.reason}'
TooManyJobsHaveFailed

>> kubectl get calls group-a -o jsonpath='{.status.message}'
tolerate: 1. failed: 2 ([group-a-2 group-a-3])
```


The `.status.message` gives us two valuable information.

* The number of failed resources is more than those tolerated
* The names of the failed resources



#### Step 2. Inspect the Virtual Objects

Using the above information, we can now dive into the virtual object and inspect the source of the failure..

```bash
>> kubectl get virtualobjects group-a-2 -o jsonpath='{.status.message}'
call 'idle-0/{app [cowsay I am A]}' error: Mismatched stdout. Expected 'oops ..' but got ' ________ 
< I am A >
 -------- 
    \
     \
      \     
                    ##        .            
              ## ## ##       ==            
           ## ## ## ##      ===            
       /""""""""""""""""___/ ===        
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
       \______ o          __/            
        \    \        __/             
          \____\______/   
'
```

