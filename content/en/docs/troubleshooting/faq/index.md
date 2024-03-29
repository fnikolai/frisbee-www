---
title: Frequently Asked Questions
linktitle: FAQ
description: Answers to frequent questions.
date: 2022-06-28
publishdate: 2022-06-28
lastmod: 2022-06-28
layout: single
menu:
    docs:
        parent: "troubleshooting"
        weight: 10
weight: 10
sections_weight: 10
draft: false
aliases: [/faq/]
toc: true
---

# Deployment

## Q: The service seems fine, but I get a Failed message.
The service run in a Pod that may host multiple containers. The application containers, the telemetry container,
and so on. Given that, if the application seems fine, it is perhaps one of the sidecar containers that has failed.

## Q: I changed some templates, but the changes does not seem to affect the Test Plan.
The changes are local and must be posted to the Kubernetes API. To update all templates within a chart use:

```bash
>> helm  upgrade --install --wait my-example ./charts/example --debug
```



## Q: Missing chart directory.
It is possible to get `helm.go:81: [debug] found in Chart.yaml, but missing in charts/ directory:` when trying to run an experiment. This may happen if the dependencies of a chart (aka subcharts) are not installed. To fixed it, simply update the
chart. for the chart `platform` (aka Frisbee), do the following

```bash
>> helm dependency update charts/platform/
```


## Q: Pods don't have internet access
This may happen either because you have not enabled DNS on microk8s (`microk8s enable dns`) or because your
firewall is blocking DNS traffic (`sudo ufw allow out to any port 53`). If non of the above work, retry to reboot you
machine !

https://github.com/canonical/microk8s/issues/1484

https://stackoverflow.com/questions/62664701/resolving-external-domains-from-within-pods-does-not-work

**Disable your firewall**

* DNS issues will manifest as a `[UFW BLOCK]... ` entries on the `dmesg`.
* If you see that, disable your firewall via `sudo ufw disable`


## Q: Ingress is not working
Ingress issues will appear as 404 HTTP codes when trying to access the HTTP page.

If so, run `kubectl describe ingress | grep "dashboard:"`  and access the HTTP page via `ip:port` scheme (e.g,
10.1.128.37:8443)

If the page is accessible then **restart your machine** and retry.


## Q: My experiment was running perfectly a few hours ago. Now, nothing works.
A possible explanation is that you do not specify the container version. If so, the latest version is retrieved
for each run. And if there are incompatibilities between version, these incompatibilities will be reflected to your
experiment.

# Grafana

## Q: All I see in Grafana is a dot. There are no lines
This is likely to happen when the duration of the experiment is too short. In general, we use 1/4 resolution in
order to make Grafana plots more readable. If you wish for a greater granularity, you can edit the chart and change
resolution to 1/1.


## Q: My plots in Grafana are not in line. The times are different
This is likely to happen if you have a change the resolution of one graph, without changing the other. Go and set
the same resolution everywhere.


## Q: I see dead plots ... on Grafana
Kubernetes v.1.22 drops support of cgroups v1 in favor of cgroup v2 API. If you run Kubernetes 1.22 make
sure that you have set `systemd.unified_cgroup_hierarchy=1` in the grub configuration.
https://rootlesscontaine.rs/getting-started/common/cgroup2/
