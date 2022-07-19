# nginx-ingress

## Installation

Helm chart [https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

* using nlb
* internal and external deployment

## Non-zero downtime setup

With this setup, there is an issue when terminating existing instances - downtime of up to 30 seconds. This is due to nlb issues with health checks.

### nginx-ingress internal

[https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

{% code title="values.yaml" %}
```
vacontroller:
  config:
    keep-alive: 10
    use-forwarded-headers: "true"
    use-proxy-protocol: "false"
  resources:
    limits:
      cpu: 2
      memory: 400Mi
    requests:
      cpu: 100m
      memory: 400Mi
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 11
  electionID: ingress-internal-controller-leader
  ingressClass: ingress-internal
  ingressClassByName: true
  ingressClassResource:
    name: ingress-internal
    enabled: true
    default: false
    controllerValue: "k8s.io/ingress-internal"
  existingPsp: eks.privileged
  podAnnotations:
    cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
    traffic.sidecar.istio.io/includeInboundPorts: ""
  podLabels:
    sidecar.istio.io/inject: "true"
  service:
    enableHttp: false
    externalTrafficPolicy: Local
    targetPorts:
      http: http
      https: http
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
      service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
      service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "10"
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: 3600
      service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: "2"
      service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "2"
      service.beta.kubernetes.io/aws-load-balancer-internal: true
      service.beta.kubernetes.io/aws-load-balancer-ssl-cert: <ssl-cert-arn>
      service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: ELBSecurityPolicy-TLS-1-2-2017-01
      service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
      service.beta.kubernetes.io/aws-load-balancer-type: nlb
podSecurityPolicy:
  enabled: true
```
{% endcode %}

### nginx-ingress external

[https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

{% code title="values.yaml" %}
```
controller:
  config:
    keep-alive: 10
    use-forwarded-headers: "true"
    use-proxy-protocol: "false"
  resources:
    limits:
      cpu: 2
      memory: 400Mi
    requests:
      cpu: 100m
      memory: 400Mi
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 11
  electionID: ingress-external-controller-leader
  ingressClass: ingress-external
  ingressClassByName: true
  ingressClassResource:
    name: ingress-external
    enabled: true
    default: false
    controllerValue: "k8s.io/ingress-external"
  existingPsp: eks.privileged
  podAnnotations:
    cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
    traffic.sidecar.istio.io/includeInboundPorts: ""
  podLabels:
    sidecar.istio.io/inject: "true"
  service:
    enableHttp: false
    externalTrafficPolicy: Local
    targetPorts:
      http: http
      https: http
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
      service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
      service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "10"
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: 3600
      service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: "2"
      service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "2"
      service.beta.kubernetes.io/aws-load-balancer-internal: false
      service.beta.kubernetes.io/aws-load-balancer-ssl-cert: <ssl-cert-arn>
      service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: ELBSecurityPolicy-TLS-1-2-2017-01
      service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
      service.beta.kubernetes.io/aws-load-balancer-type: nlb
podSecurityPolicy:
  enabled: true
```
{% endcode %}

## \[TODO] Zero downtime setup

This setup requires installing AWS Load Balancer Controller [https://kubernetes-sigs.github.io/aws-load-balancer-controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller)

### AWS Load Balancer Controller

[https://artifacthub.io/packages/helm/aws/aws-load-balancer-controller](https://artifacthub.io/packages/helm/aws/aws-load-balancer-controller)

```
clusterName: <eks-cluster-name>
serviceAccount:
  create: true
  name: aws-load-balancer-controller
  annotations:
    eks.amazonaws.com/role-arn: <iam-role-arn>
```

### nginx-ingress internal

[https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

```
controller:
  config:
    keep-alive: 10
    use-forwarded-headers: "true"
    use-proxy-protocol: "true"
  lifecycle:
    preStop:
      exec:
        command:
          - sh
          - -c
          - sleep 120
  resources:
    limits:
      cpu: 2
      memory: 400Mi
    requests:
      cpu: 100m
      memory: 400Mi
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 11
  electionID: ingress-internal-controller-leader
  ingressClass: ingress-internal
  ingressClassByName: true
  ingressClassResource:
    name: ingress-internal
    enabled: true
    default: false
    controllerValue: "k8s.io/ingress-internal"
  existingPsp: eks.privileged
  podAnnotations:
    traffic.sidecar.istio.io/includeInboundPorts: ""
  podLabels:
    sidecar.istio.io/inject: "true"
  service:
    enableHttp: false
    externalTrafficPolicy: Local
    targetPorts:
      http: http
      https: http
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: 3600
      service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
      service.beta.kubernetes.io/aws-load-balancer-scheme: "internal"
      service.beta.kubernetes.io/aws-load-balancer-type: "external"
      service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"
      service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
      service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: deregistration_delay.timeout_seconds=30,deregistration_delay.connection_termination.enabled=true
      service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: "2"
      service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "2"
      service.beta.kubernetes.io/aws-load-balancer-ssl-cert: <ssl-cert-arn>
      service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: ELBSecurityPolicy-TLS-1-2-2017-01
      service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
podSecurityPolicy:
  enabled: true
```

### nginx-ingress external

[https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

```
controller:
  config:
    keep-alive: 10
    use-forwarded-headers: "true"
    use-proxy-protocol: "true"
  lifecycle:
    preStop:
      exec:
        command:
          - sh
          - -c
          - sleep 120
  resources:
    limits:
      cpu: 2
      memory: 400Mi
    requests:
      cpu: 100m
      memory: 400Mi
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 11
  electionID: ingress-external-controller-leader
  ingressClass: ingress-external
  ingressClassByName: true
  ingressClassResource:
    name: ingress-external
    enabled: true
    default: false
    controllerValue: "k8s.io/ingress-external"
  existingPsp: eks.privileged
  podAnnotations:
    traffic.sidecar.istio.io/includeInboundPorts: ""
  podLabels:
    sidecar.istio.io/inject: "true"
  service:
    enableHttp: false
    externalTrafficPolicy: Local
    targetPorts:
      http: http
      https: http
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: 3600
      service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
      service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
      service.beta.kubernetes.io/aws-load-balancer-type: "external"
      service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"
      service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
      service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: deregistration_delay.timeout_seconds=30,deregistration_delay.connection_termination.enabled=true
      service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: "2"
      service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "2"
      service.beta.kubernetes.io/aws-load-balancer-ssl-cert: <ssl-cert-arn>
      service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: ELBSecurityPolicy-TLS-1-2-2017-01
      service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
podSecurityPolicy:
  enabled: true
```
