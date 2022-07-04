---
title: Parameters
linktitle: Parameters
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "getting-started"
        weight: 12
weight: 12
sections_weight: 12
draft: false
toc: true
---



Let's look at a slightly more complex workflow spec with parameters.



## Parameterized  Template

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Template				
metadata:
  name: whalesay			
spec:
  # invoke the whalesay template with
  # "hello world" as the default argument
  # to the message parameter
  inputs:					
    parameters:
      message: "hello-world"
  service:
    containers: 			
      - name: app
        image:  docker/whalesay 
        command: [cowsay]
        args: [{{"{{.Inputs.Parameters.message"}}]
```



This time, the `whalesay` template takes an input parameter named `message` that is passed as the `args` to the `cowsay` command. In order to reference parameters (e.g., `"{{"{{inputs.parameters.message}}"}}"`), the parameters must be enclosed in double set of brackets to escape the curly braces in YAML.



> Frisbee uses both installation time and runtime evaluated templated. The outer brackers `{{...}}` are evaluated by Helm at installation time. The inner brackers `{{"{{..}}"}}` are evaluated by the Frisbee controller at runtime. 



## Parameterized Workflow

Workflow parameters can be used to override the default parameters of the templates. The spec below demonstrates a scenario with two `whalesay` services, one invoked with parameters and one without. 

```yaml
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: hello-world
spec:
  actions:
    - action: Service
      name: default-parameters
      service:
        templateRef: whalesay  
  
    - action: Service
      name: custom-parameters
      service:
        templateRef: whalesay
        inputs:
          - { message: "Thanks for all the fish!" }
```



