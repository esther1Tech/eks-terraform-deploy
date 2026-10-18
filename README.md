# eks-terraform-deploy

Terraform configuration that builds an Amazon EKS cluster on AWS, with a Jenkins pipeline to plan, approve and apply it. The code is split into modules so that the network, the cluster and the ingress controller can be changed independently.

> Personal hands-on project. Based on a project by [ndiforfusi](https://github.com/ndiforfusi/eks-terraform-deploy), with course material as background.

## What it creates

| Module | Purpose |
|---|---|
| `modules/vpc` | VPC (`10.0.0.0/16`) with public and private subnets, built on the `terraform-aws-modules/vpc` module |
| `modules/eks-cluster` | EKS cluster and managed node groups (instance types `m5.xlarge`, `m5.large`, `t3.medium`; 1 to 10 nodes, 2 desired), plus three Kubernetes namespaces: `analytics`, `directory` and `gateway` |
| `modules/aws-alb-controller` | AWS Load Balancer Controller installed with Helm, with an IAM role for its service account (IRSA) |

The root module (`main.tf`) wires the three together. State is stored remotely in S3 (`backend.tf`).

## Pipeline

`eks-terraform-deploy/Jenkinsfile` runs four stages:

1. **Terraform init**
2. **Terraform plan**
3. **Manual approval**: a person must choose *Approve* before anything is applied
4. **Terraform deploy**: applies (or destroys, chosen by the `deploy_choice` parameter), starting with `module.vpc` and `module.eks` and then the rest

The default region is `us-west-2`.

## Use it

```bash
cd eks-terraform-deploy
terraform init
terraform plan
terraform apply
```

Before you run it:
- The state bucket name in `backend.tf` is a placeholder (`example-terraform-backend-bucket`). Create a bucket of your own and change it.
- The IAM role in `variables.tf` and the account ID in the ALB controller module (`123456789012`) are placeholders. Use your own.
- The notification addresses in the `Jenkinsfile` are `you@example.com`.
- An EKS cluster, NAT gateways and load balancers cost money while they run. Run `terraform destroy` when you finish.
