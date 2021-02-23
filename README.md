# helm-aws-load-balancer-controller
Helm chart for setting up AWS load balancer controller in your EKS cluster to provision LB

# Usage

```bash
helm upgrade -i aws-load-balancer-controller . -n platform --values=stages/shared-values.yaml --values=stages/prod/prod-values.yaml
```
