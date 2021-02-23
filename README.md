# helm-aws-load-balancer-controller
Helm chart for setting up AWS load balancer controller in your EKS cluster to provision LB

## Pre-requisites

### Namespace

Create a new namespace `platform` where we will install the `aws-load-balancer-controller` service.

```bash
kubectl create namespace platform
```

### IAM

We will be using IRSA (IAM Roles for Service Accounts) to give the required permissions to the AWS Load Balancer Controller pod to provision load balancers.

1. Create a new IAM policy `aws-load-balancer-controller-pol` with the policy document at `iam/policy.json`

2. Create a new IAM role `aws-load-balancer-controller-rol` and attach the IAM policy `aws-load-balancer-controller-pol`

3. Update the trust relationship of the IAM role `aws-load-balancer-controller-rol` as below replacing the `account_id`, `eks_cluster_id` and `region` with the appropriate values.

This trust relationship allows pods with serviceaccount `aws-load-balancer-controller` in `platform` namespace to assume the role.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account_id>:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/<eks_cluster_id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.<region>.amazonaws.com/id/<eks_cluster_id>:sub": "system:serviceaccount:platform:aws-load-balancer-controller"
        }
      }
    }
  ]
}
```

### Service Account

Create a new service account in the `platform` namespace and associate it with the IAM role which we had created earlier.

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-load-balancer-controller
  namespace: platform
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<AWS_ACCOUNT_ID>:role/aws-load-balancer-controller-rol
EOF
```

We specify the service account to be used by the pods in the file `stages/shared-values.ymal`

### Config Updates

1. Update `stages/prod/prod-values.yaml` file with EKS cluster and VPC Id.

```yaml
clusterName: # Your EKS cluster name here
vpcId: # Your VPC id here
```

## Usage

Run below helm command to install/upgrade the helm chart by providing shared and stage specific values.

```bash
helm upgrade -i aws-load-balancer-controller . -n platform --values=stages/shared-values.yaml --values=stages/prod/prod-values.yaml
```
