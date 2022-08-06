---
title: Time-Driven Scheduling
linktitle: Time-Driven Scheduling
description: Learn how to spread the creation of services over time
date: 2022-06-29
publishdate: 2022-06-29
authors: [Fotis NIKOLAIDIS]
menu:
    docs:
        parent: "walkthrough"
        weight: 16
weight: 16
sections_weight: 16
categories: [testing,fundamentals]
draft: false
toc: true
---



Users may also set the creation policy to construct variable workloads and dynamically changing topologies for elastic experiments. 



## Time-Driven Scheduling 

The next snippet shows how to schedule the creation of new services, using a `cron-like syntax`.

```yaml
---
apiVersion: frisbee.dev/v1alpha1
kind: Scenario
metadata:
  name: scheduling
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
          - { target: server, duration: "10" }
          - { target: server, duration: "20" }
          - { target: server, duration: "30" }
        # Schedule the creation of new clients
        schedule: 
          cron: "@every 1m"
```



All that it takes, it to add the `scheduling` fields and specify the desired cron, and you will get an output like this one:

```bash
>> kubectl logs server
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 0.12 MByte (default)
------------------------------------------------------------
[  4] local 10.244.1.214 port 5001 connected with 10.244.4.175 port 36362
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0- 5.0 sec   411 MBytes   690 Mbits/sec
[  4]  5.0-10.0 sec   408 MBytes   684 Mbits/sec
[  4]  0.0-10.0 sec   821 MBytes   687 Mbits/sec
[  4] local 10.244.1.214 port 5001 connected with 10.244.4.176 port 36562
[  4]  0.0- 5.0 sec   416 MBytes   697 Mbits/sec
[  4]  5.0-10.0 sec   412 MBytes   691 Mbits/sec
[  4] 10.0-15.0 sec   411 MBytes   690 Mbits/sec
[  4] 15.0-20.0 sec   415 MBytes   696 Mbits/sec
[  4]  0.0-20.0 sec  1654 MBytes   693 Mbits/sec
[  4] local 10.244.1.214 port 5001 connected with 10.244.4.177 port 36010
[  4]  0.0- 5.0 sec   406 MBytes   681 Mbits/sec
[  4]  5.0-10.0 sec   409 MBytes   685 Mbits/sec
[  4] 10.0-15.0 sec   408 MBytes   685 Mbits/sec
[  4] 15.0-20.0 sec   407 MBytes   682 Mbits/sec
[  4] 20.0-25.0 sec   397 MBytes   666 Mbits/sec
[  4] 25.0-30.0 sec   411 MBytes   690 Mbits/sec
[  4]  0.0-30.0 sec  2438 MBytes   682 Mbits/sec
```

