---
title: Assertions
linktitle: Assertions
description: Assert the behavior of a running system.
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


No testing framework can be complete without assertions.

Let's see how to use them properly.


In the following scenario, we:

* Run a server that terminates gracefully after a few seconds

* Run a set of clients

* Evaluate the outcome with and without assertions.


## Assertions


#### Template for gracefully terminated server

Firstly, let's make a server that gracefully terminates after a few seconds.

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: iperf.server
spec:
  service:
    containers:
      - name: app
        image: czero/iperf2
        command:
          - /bin/sh   # Run shell
          - -c        # Read from string
          - |         # Multi-line str
            # Run the iperf server in the background
            iperf -s -f -m -i 5 &
            
            # Gracefully terminate after the given period
            sleep 60 && exit 0
```


#### Scenario, Without Assertions

Then, let's run our usual scenario.

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: assertions
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
        inputs:
          - { target: server, duration: "400" }
          - { target: server, duration: "400" }
          - { target: server, duration: "400" }
```


After around 60 seconds, if you check the status of the scenario you will see that the experiment completed successfully.

```bash
>> kubectl get scenarios assertions  -o jsonpath='{.status.phase}'
Success
```


This is because after the server's termination, `the clients detected the broken connection and they exit themselves (gracefully)`.  It makes sense, but is that the desired behavior?


#### Scenario, With Assertions

Let's try repeat the previous scenario, but now let's use assertions.


```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: assertions
spec:
  actions:
    - action: Service
      name: server
      service:
        templateRef: iperf.server

    - action: Cluster
      name: clients
      depends: { running: [ server ] }  # Depends are pre-execution conditions
      assert:                           # Assert are post-execution conditions
        state: '{{.IsRunning "server"}}'
      cluster:
        templateRef: iperf.client
        inputs:
          - { target: server, duration: "400" }
          - { target: server, duration: "400" }
          - { target: server, duration: "400" }
```


After around 60 seconds, if you check the status of the scenario you will see that the experiment completed successfully.

```bash
>> kubectl get scenarios assertions  -o jsonpath='{.status.phase}'
Failed
```


This is because after the server's termination, `the Scenario detects that the server is missing and aborts (with a failure)`.


## Assertion's Scope


Let's try to repeat the `conditional` scenario we used before.

But, in this case, along with the `until` condition which `suspends` the scenario when the condition is met, we have added an `assert` to `abort` the scenario when the condition is met.


```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
	name: assertions-scope
spec:
  actions:
    - action: Service
      name: server
      service:
      templateRef: iperf.server

	# Scenario Scope
    - action: Cluster
      name: clients
      depends: { running: [ server ] }
      assert: # Abort the scenario
        state: '{{.NumSuccessfulJobs}} >= 4'
      # Cluster Scope
      cluster:
        templateRef: iperf.client
        instances: 10
        inputs:
          - { target: server, duration: "10" }
          - { target: server, duration: "20" }
          - { target: server, duration: "30" }
         schedule:
           cron: "@every 1m"
         until: # Suspend the cluster
           state: '{{.NumSuccessfulJobs}} >= 6'
```


You wait, and wait, and wait, .... and the scenario keep creating clients.

What happened ?


#### State expressions are locally scoped

As we explain previously for  `until`:,

* the expression `{{.NumSuccessfulJobs}} >= 6` is **local** to the `Cluster` and refers to the jobs managed by the `Cluster `controller.

Equally for the `assertion`,

* the expression`'{{.NumSuccessfulJobs}} >= 4'` is **local** to the `Scenario` and refers to the jobs managed by the `Scenario` controller.


In other words, the `Scenario` accounts only for `Actions`, and not for jobs created within the context of a `Cluster`. The reason behind this convience will become clear later when we talk about `Cluster.Spec.Tolerate.FailedJobs`.





