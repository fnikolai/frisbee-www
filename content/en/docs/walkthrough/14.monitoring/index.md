---
title: Live Monitoring
linktitle: Live Monitoring
description: The Live Monitoring feature provides authorized users with the ability to monitor live interactions within the SUT.
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
  docs:
    parent: "walkthrough"
    weight: 23
weight: 23
sections_weight: 23
categories: [testing,fundamentals]
draft: false
toc: true
---





## Resource Monitoring

Frisbee integrates the following web-based dashboard which consumes information from the Kubernetes API.

* [Dashboard](https://dashboard-frisbee.localhost/#/pod?namespace=mytest):  is a web-based Kubernetes user interface.
  You can use Dashboard to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and manage the cluster resources.

* [Chaos Dashboard](http://chaos-frisbee.localhost/experiments): is a one-step web UI for managing, designing, and
  monitoring chaos experiments on *Chaos Mesh*. It will ask for a token. You can get it from the config
  via `grep token  ~/.kube/config`.



To see the endpoints for accessing the dashboards do: 

```bash
>> kubectl get ingress
NAME              CLASS    HOSTS                          ADDRESS     PORTS   AGE
chaos-dashboard   public   chaos-frisbee.localhost        127.0.0.1   80      47h
dashboard         public   dashboard-frisbee.localhost    127.0.0.1   80      47h
grafana           public   grafana-default.localhost      127.0.0.1   80      104s
prometheus        public   prometheus-default.localhost   127.0.0.1   80      104s
```





## Performance Monitoring

Frisbee offers a set of  user-friendly web interfaces through which  users can observe the experiments as they run. The performance monitoring stack consists of:

* **Telemetry Agents**: running on the Pod so to collect and expose performance metrics to Prometheus.

* **Prometheus:** Collect metrics from the distributed agents and store them in a time-series database

* **Grafana:** Provides a Visualization Frontend to Prometheus, along with alerting capabilities.

  

Adding support for Telemetry generally consists of four steps.

1. Define the template for a `telemetry agent`.
2. Deploy the telemetry agent as `sidecar` into the Pod of a `Service`.
3. Define a `Grafana Dashboard` that understand and can consume the collected metrics.
4. Deploy the Grafana Dashboard into a running `Grafana Instance`.



All the above are cumbersome, and we need to automate them as much as possible. Let's how we provide out-of-the-box monitoring in Frisbee.



#### Declare Telemetry Packages to Use.

To support Telemetry capabilities, we need somehow to install the Telemetry agents into the running Pods. Normally, this is a rather involved process. With Frisbee, it is straight-forward using the concept of `Decorators` and `sidecars`. 



```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: iperf.server
spec:
  service:
    decorators:	# Add support for Telemetry
      telemetry: [system.telemetry.agent]
    containers:
      - name: app
        image: czero/iperf2
        command: [ iperf ]
        args: [ "-s", "-f", "m", "-i", "5" ]

---
apiVersion: frisbee.dev/v1alpha1
kind: Template
metadata:
  name: iperf.client
spec:
  inputs:
    parameters:
      target: localhost
  service:
    decorators:						
      telemetry:				  # Add support for Telemetry
        - system.telemetry.agent  # Collect Generic System's Metrics
        - iperf2.telemetry.client # Collect App Specific Metrics
    containers:
      - name: app
        image: czero/iperf2
        command: [ iperf ]
        args: [ "-c", "{{.Inputs.Parameters.target}}", "-t", "500" ]
```



In the previous experiment, we used two Telemetry agents that exist on different `Helm Charts` that must be installed.

```bash
# Install system-wise telemetry agents (system.telemetry.agent)
helm upgrade --install  my-system ./charts/system/  --debug --wait
# Install app-specic telemetry agents (iperf2.telemetry.client)
helm upgrade --install  my-iperf2 ./charts/iperf2/  --debug --wait
```



You should notice a few things:

1. All that it takes to support telemetry, is to add a reference to the template that implements the telemetry agent.
2. Every template can have its own combination of telemetry agents.
3. You may have generic agents that collect system-wise information (e.g, CPU, Mem, Disk, Net) or app-specific metrics that collect internal information (e.g, goroutines, requests/sec,). 
4. The generic agents can be used for `blackbox testing` , whereas the app-specific agents for `greybox testing`.
5. There is no need to modify the Scenario.
