# Loki Operator Development Guide

This file provides detailed architectural guidance and development context for AI agents working with the Loki Operator codebase.

## Overview

The Loki Operator is a Kubernetes controller that manages Loki deployments using Custom Resource Definitions (CRDs). It provides a declarative approach to deploying and managing Loki in Kubernetes and OpenShift environments.

## Directory Structure

**`operator/`**: the operator sub-project

- **`api/`**: Custom Resource Definitions and API types
  - `loki/v1/`: Stable API version with core resource types
    - `lokistack_types.go`: Defines LokiStack custom resource for complete Loki deployments
    - `alertingrule_types.go`: Manages LogQL-based alerting rules
    - `recordingrule_types.go`: Defines recording rules for pre-computed metrics
    - `rulerconfig_types.go`: Configures the ruler component behavior
  - `loki/v1beta1/`: Beta API version for experimental features

- **`cmd/`**: Operator executables and utilities
  - `loki-operator/`: Main operator controller binary
  - `loki-broker/`: CLI tool for operator management and debugging
  - `size-calculator/`: Storage size calculation utility for capacity planning

- **`internal/`**: Core operator implementation (not part of public API)
  - `controller/`: Kubernetes controller reconciliation logic
  - `config/`: Configuration management and validation
  - `manifests/`: Kubernetes manifest generation and templating
  - `operator/`: Core operator business logic and resource management
  - `validation/`: Resource validation and admission control
  - `sizes/`: Storage sizing algorithms and calculations

- **`config/`**: Kubernetes deployment configurations
  - `crd/`: Custom Resource Definition bases
  - `rbac/`: Role-Based Access Control configurations
  - `manager/`: Operator deployment manifests
  - `samples/`: Example Custom Resource configurations
  - `kustomize/`: Kustomize overlays for different environments

- **`bundle/`**: Operator Lifecycle Manager (OLM) packaging
  - Supports multiple deployment variants:
    - `community`: Standard Grafana distribution
    - `community-openshift`: OpenShift-compatible community version
    - `openshift`: Red Hat certified OpenShift distribution
  - Contains ClusterServiceVersion, package manifests, and bundle metadata

## Key Features

- **Multi-tenant Support**: Isolates log streams by tenant with configurable authentication
- **Flexible Storage**: Supports object storage (S3, GCS, Azure), local storage, and hybrid configurations
- **Auto-scaling**: Horizontal Pod Autoscaler integration for dynamic scaling
- **Security**: Integration with OpenShift authentication, RBAC, and network policies
- **Monitoring**: Built-in Prometheus metrics and Grafana dashboard integration
- **Gateway Component**: Optional log routing and tenant isolation layer

## Deployment Variants

1. **Community** (`VARIANT=community`):
   - Registry: `docker.io/grafana`
   - Standard Kubernetes deployment
   - Flexible configuration options
   - Community support channels

2. **Community-OpenShift** (`VARIANT=community-openshift`):
   - Optimized for OpenShift but community-supported
   - Enhanced security contexts
   - OpenShift-specific networking configurations

3. **OpenShift** (`VARIANT=openshift`):
   - Registry: `quay.io/openshift-logging`
   - Namespace: `openshift-operators-redhat`
   - Full Red Hat support and certification
   - Tight integration with OpenShift Logging stack

## Build and Development Commands

```bash
# Core Development Workflow
make generate                   # Generate controller and CRD code
make manifests                  # Generate CRDs, RBAC, and deployment manifests
make fmt                        # Format Go source code
make vet                        # Run Go vet static analysis
make test                       # Execute unit tests
make lint                       # Run comprehensive linting

# Local Development and Testing
make run                        # Run operator locally against configured cluster
make deploy                     # Deploy operator to cluster via kubectl
make undeploy                   # Remove operator from cluster
make install                    # Install CRDs to cluster
make uninstall                  # Remove CRDs from cluster

# Container and Bundle Management
make docker-build               # Build operator container image
make docker-push                # Push operator image to registry
make bundle                     # Generate OLM bundle for specified variant
make bundle-push                # Push bundle to registry
make catalog-build              # Build catalog image for OLM
make catalog-push               # Push catalog image

# Advanced Deployment Options
make olm-deploy                 # Deploy via Operator Lifecycle Manager
make scorecard                  # Run OLM scorecard tests
make quickstart                 # Set up local development environment
```

## Testing Strategy

```bash
# Unit Testing
make test                       # Run all unit tests
go test ./internal/...          # Test internal packages
go test ./api/...               # Test API types and validation

# Integration Testing
make e2e                        # Run end-to-end tests (requires cluster)
make test-storage               # Test storage backend integrations
make test-openshift             # OpenShift-specific integration tests

# Quality Assurance
make lint-rules                 # Validate Prometheus alerting rules
make bundle-validate            # Validate OLM bundle structure
make scorecard                  # Run OLM certification tests
```

## Contributing to Operator Code

1. **Development Environment Setup**:
   ```bash
   # Prerequisites: Go 1.21+, Docker/Podman, kind or OpenShift cluster
   git clone https://github.com/grafana/loki.git
   cd loki/operator
   make quickstart                # Sets up local environment
   ```

2. **Development Workflow**:
   ```bash
   # Make changes to API types, controllers, or manifests
   make generate manifests       # Regenerate code and manifests
   make fmt vet lint             # Ensure code quality
   make test                     # Run unit tests
   make run                      # Test locally against cluster
   ```

3. **Adding New API Fields**:
   - Modify types in `api/loki/v1/*.go`
   - Run `make generate manifests` to update generated code
   - Add validation logic in `internal/validation/`
   - Update controller reconciliation in `internal/controller/`
   - Write comprehensive unit tests

4. **Adding New Features**:
   - Extend controller logic in `internal/controller/lokistack/`
   - Add manifest generation in `internal/manifests/`
   - Update configuration handling in `internal/config/`
   - Add feature flags if needed
   - Document in operator documentation

5. **Testing Your Changes**:
   ```bash
   # Local testing workflow
   make install                  # Install CRDs
   make run                      # Run operator locally
   # Apply sample CustomResources in another terminal
   kubectl apply -f config/samples/
   ```

6. **Bundle and Release Process**:
   ```bash
   # Test bundle generation for all variants
   make bundle VARIANT=community
   make bundle VARIANT=community-openshift
   make bundle VARIANT=openshift
   make bundle-validate          # Validate all bundles
   ```

## Common Development Tasks

- **Adding New CRD Field**: Modify `*_types.go`, run `make generate manifests`
- **Updating Controller Logic**: Edit `internal/controller/`, ensure proper reconciliation
- **Adding Storage Backend**: Extend `internal/manifests/storage.go`
- **Enhancing Validation**: Update `internal/validation/` with new rules
- **Supporting New Loki Version**: Update manifests and test compatibility

## Troubleshooting Development Issues

```bash
# Debug operator logs
kubectl logs -f deployment/loki-operator-controller-manager -n loki-operator-system

# Check CRD status
kubectl describe lokistack my-stack -n my-namespace

# Validate generated manifests
kubectl apply --dry-run=client -f config/samples/

# Test bundle locally
operator-sdk run bundle-upgrade docker.io/grafana/loki-operator-bundle:latest
```
