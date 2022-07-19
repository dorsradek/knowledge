# Getting started

## Docs

## Installation

The simplest is using the Helm chart.

### base

[https://artifacthub.io/packages/helm/istio-official/base](https://artifacthub.io/packages/helm/istio-official/base)

{% code title="values.yaml" %}
```
// no values required
```
{% endcode %}

### cni

[https://artifacthub.io/packages/helm/istio-official/cni](https://artifacthub.io/packages/helm/istio-official/cni)

{% code title="values.yaml" %}
```yaml
cni:
  psp_cluster_role: psp-istio-cni-node
  excludeNamespaces:
    - istio-system
    - kube-system
```
{% endcode %}

### istiod

[https://artifacthub.io/packages/helm/istio-official/istiod](https://artifacthub.io/packages/helm/istio-official/istiod)

{% code title="values.yaml" %}
```yaml
istio_cni:
  enabled: true
meshConfig:
  accessLogEncoding: JSON
  accessLogFile: /dev/stdout
  defaultConfig:
    concurrency: 0
    holdApplicationUntilProxyStarts: true
    terminationDrainDuration: 30s
    proxyMetadata:
      EXIT_ON_ZERO_ACTIVE_CONNECTIONS: 'true'
  enableTracing: true
  outboundTrafficPolicy:
    mode: ALLOW_ANY
pilot:
  autoscaleMin: 2
  autoscaleMax: 5
  keepaliveMaxServerConnectionAge: 30m
  resources:
    limits:
      cpu: 200m
      memory: 500Mi
    requests:
      cpu: 50m
      memory: 500Mi
  traceSampling: 100
global:
  logAsJson: true
  proxy:
    logLevel: info
    resources:
      limits:
        cpu: 400m
        memory: 400Mi
      requests:
        cpu: 100m
        memory: 100Mi
    tracer: datadog
```
{% endcode %}
