To create EKS Cluster

# Create cluster

```
eksctl create cluster -p prod_sts --config-file=eks.cluster.yaml
```

# Install CNI

Install https://github.com/aws/amazon-vpc-cni-k8s

```
kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/release-1.7/config/v1.7/aws-k8s-cni.yaml
```

# Install kube2iam

Install https://github.com/jtblin/kube2iam

```
kubectl apply -f kube2iam.yaml
```

# Configure IAM

1. Go to https://console.aws.amazon.com/iam/home?region=us-east-2 and create a Role for your K8S service with name `{Service}.service`
   Copy the `Trusted Relationship` from https://console.aws.amazon.com/iam/home?region=us-east-2#/roles/Opa.Service?section=trust

The snipped should have Worker Node

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

2. Go to the https://console.aws.amazon.com/iam/home?region=us-east-2#/policies/arn:aws:iam::903635927707:policy/StsAssumeRole$serviceLevelSummary and add there Allowed Roles to assume

```

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

Allow EBS expansion

```
k patch sc gp2 -p '{"allowVolumeExpansion": true}'
```
