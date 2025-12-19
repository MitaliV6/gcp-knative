# Knative Helm Chart

A Helm chart for deploying Knative Serving and Eventing resources.

## Features

- **Knative Serving**: Deploy serverless services with auto-scaling
- **Knative Eventing**: Set up event-driven architectures with brokers, triggers, and sources
- **Flexible Configuration**: Support for multiple Knative resources
- **Auto-scaling**: Built-in support for Knative auto-scaling annotations

## Prerequisites

- Kubernetes cluster with Knative Serving and/or Eventing installed
- Helm 3.x
- kubectl configured to access your cluster

## Installation

### Basic Installation

```bash
helm install my-knative ./charts/knative
```

### With Custom Values

```bash
helm install my-knative ./charts/knative -f my-values.yaml
```

## Configuration

### Knative Serving

#### Service

The main Knative Service resource:

```yaml
serving:
  enabled: true
  service:
    enabled: true
    name: my-service
    template:
      spec:
        containers:
          - name: user-container
            image: nginx:latest
            ports:
              - containerPort: 80
```

#### Autoscaling

Configure auto-scaling behavior:

```yaml
autoscaling:
  enabled: true
  minScale: 0
  maxScale: 10
  target: 100
  annotations:
    autoscaling.knative.dev/minScale: "0"
    autoscaling.knative.dev/maxScale: "10"
    autoscaling.knative.dev/target: "100"
```

#### Traffic Splitting

Split traffic between revisions:

```yaml
serving:
  service:
    traffic:
      - revisionName: my-service-00001
        percent: 80
        tag: stable
      - latestRevision: true
        percent: 20
        tag: canary
```

### Knative Eventing

#### Broker

Create an event broker:

```yaml
eventing:
  enabled: true
  broker:
    enabled: true
    name: default
    spec:
      config:
        apiVersion: v1
        kind: ConfigMap
        name: kafka-broker-config
        namespace: knative-eventing
```

#### Trigger

Create a trigger to filter events:

```yaml
eventing:
  trigger:
    enabled: true
    name: my-trigger
    broker: default
    spec:
      filter:
        attributes:
          type: dev.knative.samples.helloworld
          source: dev.knative.samples/helloworldsource
      subscriber:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: my-service
```

#### PingSource

Create a cron-like event source:

```yaml
eventing:
  source:
    ping:
      enabled: true
      name: my-ping-source
      schedule: "*/1 * * * *"
      data: '{"message": "Hello"}'
      sink:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: my-service
```

## Examples

### Example 1: Simple Knative Service

```yaml
serving:
  enabled: true
  service:
    enabled: true
    name: hello-world
    template:
      spec:
        containers:
          - name: user-container
            image: gcr.io/knative-samples/helloworld-go:latest
            ports:
              - containerPort: 8080
```

### Example 2: Service with Custom Resources

```yaml
serving:
  enabled: true
  service:
    enabled: true
    name: my-app
    template:
      spec:
        containers:
          - name: user-container
            image: my-registry/my-app:latest
            resources:
              limits:
                cpu: 1000m
                memory: 512Mi
              requests:
                cpu: 100m
                memory: 128Mi
```

### Example 3: Event-Driven Application

```yaml
serving:
  enabled: true
  service:
    enabled: true
    name: event-processor
    template:
      spec:
        containers:
          - name: user-container
            image: my-registry/event-processor:latest

eventing:
  enabled: true
  broker:
    enabled: true
    name: default
  
  trigger:
    enabled: true
    name: event-processor-trigger
    broker: default
    spec:
      subscriber:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: event-processor
```

## Values Reference

| Parameter | Description | Default |
|-----------|-------------|---------|
| `namespace` | Namespace for resources | `default` |
| `serving.enabled` | Enable Knative Serving resources | `true` |
| `serving.service.enabled` | Create Knative Service | `true` |
| `serving.service.name` | Service name | `knative-service` |
| `eventing.enabled` | Enable Knative Eventing resources | `false` |
| `eventing.broker.enabled` | Create Broker | `false` |
| `autoscaling.enabled` | Enable auto-scaling | `true` |
| `autoscaling.minScale` | Minimum number of pods | `0` |
| `autoscaling.maxScale` | Maximum number of pods | `10` |
| `serviceAccount.create` | Create service account | `true` |

## Troubleshooting

### Check Service Status

```bash
kubectl get ksvc -n <namespace>
kubectl describe ksvc <service-name> -n <namespace>
```

### Check Revision Status

```bash
kubectl get revisions -n <namespace>
kubectl describe revision <revision-name> -n <namespace>
```

### Check Route Status

```bash
kubectl get routes -n <namespace>
kubectl describe route <route-name> -n <namespace>
```

### Check Eventing Resources

```bash
kubectl get brokers -n <namespace>
kubectl get triggers -n <namespace>
kubectl get pingsources -n <namespace>
```

### View Logs

```bash
kubectl logs -l serving.knative.dev/service=<service-name> -n <namespace>
```

## Uninstallation

```bash
helm uninstall my-knative
```

## Additional Resources

- [Knative Serving Documentation](https://knative.dev/docs/serving/)
- [Knative Eventing Documentation](https://knative.dev/docs/eventing/)
- [Knative Samples](https://github.com/knative/samples)

