# Create EKS(Already Done)

## Dev

```
eksctl create cluster --config-file=eks.cluster.yaml
```

## Create cluster prod

```
eksctl create cluster -p prod_sts --config-file=eks.cluster.yaml
```

## Configure Access to AWS EKS

https://www.notion.so/pieeye/K8S-e71430e032cc4efc8703b779d8de4f07

## Install CNI

Install https://github.com/aws/amazon-vpc-cni-k8s

```
kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/release-1.7/config/v1.7/aws-k8s-cni.yaml
```

## Install kube2iam

Install https://github.com/jtblin/kube2iam

```
kubectl apply -f kube2iam.yaml
```

## Create 1 ALB (once per setup)

Follow https://aws.amazon.com/premiumsupport/knowledge-center/eks-kubernetes-dashboard-custom-path/ to create ALB ingress see the [alb-ingress.yaml](alb-ingress.yaml)

## Allow EBS expansion

```
k patch sc gp2 -p '{"allowVolumeExpansion": true}'
```

# Add a service

## Configure IAM

1. Go to https://console.aws.amazon.com/iam/home?region=us-east-2 and create a Role for your K8S service with name `{service}.service` example for K8S service `opa` create a role `opa.service`

Add a Trusted Relationship to just created role with Worker Nodes ARN roles. This allows worker nodes to Assume `{service}.service` role.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::006396608450:role/eksctl-dev-nodegroup-a1-NodeInstanceRole-1I4LVNJH23P8O"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

2. Edit `StsAssumeRole` IAM policty and add there Allowed Roles to assume

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::903635927707:role/opa.service"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::903635927707:role/dsrppublish.service"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::903635927707:role/connector-engine.service"
        },
        {
            "Sid": "VisualEditor3",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::903635927707:role/connector-scheduler.service"
        },
        {
            "Sid": "VisualEditor4",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::903635927707:role/dsrpworkflow.service"
        }
    ]
}
```

3. Go to your K8S resource and add annotation

```
annotations:
    iam.amazonaws.com/role: arn:aws:iam::903635927707:role/{Service}.service
```

## Create the services and ingress routes

```
k apply -f {servicedir}/service.yaml
k apply -f {servicedir}/nginx-ingress.yaml
```

# Deploy services

Configure access https://www.notion.so/pieeye/K8S-e71430e032cc4efc8703b779d8de4f07
Go update the {servicedir}/deployment.yaml with correct `spec.template.spec.containers.name` value.
image

```
k apply -f {servicedir}/deployment.yaml
```

# Access for SSO Roles to the EKS
```
eksctl create iamidentitymapping --group system:masters --username "admin:{{SessionName}}" --arn "arn:aws:iam::006396608450:role/AWSReservedSSO_AdministratorAccess_2218ff24a6a5a597" --cluster dev --region us-east-2
```
