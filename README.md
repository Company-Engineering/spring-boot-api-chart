<h1 align="center">Spring Boot API service Chart</h1>

Helm chart for deploying the `Spring Boot API` service to Kubernetes.

## Deploying the chart

Refer to [releasing-spring-boot-api-service](github.com/Company-Engineering/spring-boot-api-gitops/blob/main/README.md#releasing-spring-boot-api-service) section for information on detailed release process.

This repository is the packaging layer only. It is designed to be consumed by [spring-boot-api-gitops](https://github.com/Company-Engineering/spring-boot-api-gitops), where environment-specific values and promotion rules are defined.

In this setup:

- this chart provides the reusable deployment logic
- GitOps definitions decide which chart revision is deployed to each environment
- environment values control replica count, ingress host, profile, config payload, and resource sizing

That split keeps the chart generic and reusable, while still allowing environment promotion and release management through GitOps.

## Repository structure

```text
spring-boot-api-chart/
└── helm/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── _helpers.tpl
        ├── configmap.yaml
        ├── deployment.yaml
        ├── ingress.yaml
        ├── secret.yaml
        └── service.yaml
```

## What the chart does

The chart deploys a single application workload behind one Kubernetes `Service`, with separate ports for:

- `api` on `8080`
- `logs` on `8081`
- `soap` on `8082`

When ingress is enabled, those endpoints are mapped to distinct paths on a single host:

- `/api`
- `/logs`
- `/soap`

The deployment is parameterized so the same chart can be reused across environments by changing only values such as image tag, replica count, ingress host, resource sizing, and runtime profile.

## Implementation highlights

This chart covers:

- Rolling updates are enabled with `maxUnavailable: 0` and `maxSurge: 1`
- Liveness and readiness probes are configured on the application port
- A `preStop` lifecycle hook calls the configured shutdown endpoint before termination
- Runtime configuration is injected through a `ConfigMap` and mounted into the container as `/app/config.json`
- A checksum annotation is derived from the `ConfigMap`
- Application credentials are stored in a Kubernetes `Secret`
- Credentials are generated on first install and then reused on upgrades via Helm `lookup`
- Resource requests and limits are part of the default values
- Ingress support for `dev` and `prd` env

## Default configuration

Out of the box the chart ships with:

- `replicaCount: 1`
- image repository `busybox` and tag `stable`
- `ClusterIP` service exposure
- ingress disabled
- CPU and memory requests/limits already defined
- configurable Spring profile through `springProfile`
- configurable shutdown URL through `shutdown.url`

The main values intended to be overridden per environment are:

- `image.repository`
- `image.tag`
- `replicaCount`
- `springProfile`
- `ingress.enabled`
- `ingress.className`
- `ingress.host`
- `config.json`
- `resources`