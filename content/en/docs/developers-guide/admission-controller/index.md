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

```
https://tech.paulcz.net/blog/creating-self-signed-certs-on-kubernetes/
```





#### Validate the certificates against the CA

```
openssl verify -CAfile <(kubectl get secret ca-tls -o jsonpath='{.data.ca\.crt}' | base64 -d) <(kubectl get secret webhook-tls -o jsonpath='{.data.tls\.crt}' | base64 -d)
```





#### **Validate the Client / Server authentication**

```bash
# Run server
openssl s_server \
  -cert <(kubectl get secret webhook-tls -o jsonpath='{.data.tls\.crt}' | base64 -d) \
  -key <(kubectl get secret webhook-tls -o jsonpath='{.data.tls\.key}' | base64 -d) \
  -CAfile <(kubectl get secret ca-tls -o jsonpath='{.data.ca\.crt}' | base64 -d) \
  -WWW -port 12345  \
  -verify_return_error -Verify 1 &
  
  
# Run client
echo -e 'GET /test.txt HTTP/1.1\r\n\r\n' | \
  openssl s_client \
  -cert <(kubectl get secret sandbox2-client-tls -o jsonpath='{.data.tls\.crt}' | base64 -d) \
  -key <(kubectl get secret sandbox2-client-tls -o jsonpath='{.data.tls\.key}' | base64 -d) \
  -CAfile <(kubectl get secret sandbox2-client-tls -o jsonpath='{.data.ca\.crt}' | base64 -d) \
  -connect localhost:12345 -quiet
```

