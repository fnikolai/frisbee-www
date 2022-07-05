---
title:  Admission Controller
linktitle: Admission Controller
description:
date: 2022-06-28
publishdate: 2022-06-28
lastmod: 2022-06-28
layout: single
menu:
    docs:
        parent: "developers-guide"
        weight: 8
weight: 10
sections_weight: 10
draft: true
aliases: []
toc: true
---








## Admission Controller

https://docs.giantswarm.io/advanced/custom-admission-controller/



```bash
helm upgrade --install  my-frisbee ./charts/platform/ --set operator.enabled=false -f ./charts/platform/values.yaml  --debug
```





## Certificates



```bash
export CA_BUNDLE=$(cat certs/ca.crt | base64 | tr -d '\n')
helm upgrade --install  my-frisbee ./charts/platform/ --set operator.enabled=false -f ./charts/platform/values-baremetal.yaml  --debug --set operator.webhook.k8s.caBundle="$CA_BUNDLE"


# Verify the certificate
make certs
openssl s_client -connect 139.91.92.82:9443 &> /tmp/aposkata


kubectl get secret controller-certificate -o json | jq -r '.data["ca.crt"]' | base64 -d > ca.crt
kubectl get secret controller-certificate -o json | jq -r '.data["tls.crt"]' | base64 -d > tls.crt
kubectl get secret controller-certificate -o json | jq -r '.data["tls.key"]' | base64 -d > tls.key

ENABLE_WEBHOOKS="true" make run
```

