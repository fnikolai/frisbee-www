    ---
title: Hello World
linktitle: Hello World 
description: Hello-World in Frisbee
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "walkthrough"
        weight: 11
weight: 11
sections_weight: 11
categories: [testing,fundamentals]
draft: false
toc: true
---


Let's start by creating a very simple workflow template to echo "hello world" using the docker/whalesay container image from Docker Hub.



You can run this directly from your shell with a simple docker command:

```bash
$ docker run docker/whalesay cowsay "hello world"
 _____________
< hello world >
 -------------
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


Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Below, we run the same container on a Kubernetes cluster using an Frisbee workflow template. Be sure to read the comments as they provide useful explanations.



## Create an Application Template

Templates define minimally constraining skeletons for services. This strategy allows us to create a library of frequently-used specifications and use them to generate objects on-demand throughout the experiment. The spec below contains a single `template` called `whalesay` which runs the `docker/whalesay` container and invokes `cowsay "hello world"`. 

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Template				# new type of k8s spec
metadata:
  name: whalesay			# name of the template spec
spec:
  service:
    containers: 			# application container
      - name: app
        image:  docker/whalesay 
        command: [cowsay]
        args: ["hello-world"]
```



## Create a Testing Workflow

Frisbee adds a new `kind` of Kubernetes spec called a `Workflow`. 

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: hello-world
spec:
  actions:
    # create a service using the whalesay template
    - action: Service	 
      name: whalesay
      service:
        templateRef: whalesay  
```
