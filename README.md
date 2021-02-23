# helm-aws-load-balancer-controller
Helm chart for setting up AWS load balancer controller in your EKS cluster to provision LB

## Pre-requisites

1. Create a new namespace `platform` where we will install the `aws-load-balancer-controller` services.

```bash
kubectl create namespace platform
```

2. Update `stages/prod/prod-values.yaml` file with EKS cluster and VPC Id.

```yaml
clusterName: # Your EKS cluster name here
vpcId: # Your VPC id here
```

## Usage

Run below helm command to install/upgrade the helm chart by providing shared and stage specific values.

```bash
helm upgrade -i aws-load-balancer-controller . -n platform --values=stages/shared-values.yaml --values=stages/prod/prod-values.yaml
```
