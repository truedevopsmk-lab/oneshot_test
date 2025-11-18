# Oneshot Cluster Creation Scripts
> Making it Super simple with just one template file that uses predetermined/existing default subnet.  

## Files
- cluster-template.yaml — template using env vars
- create.sh — generates YAML + creates cluster
- delete.sh — deletes cluster

## Usage
Create:
```
./create.sh PRE-3
```

Delete:
```
./delete.sh PRE-3
```


## cluster-template.yaml just for Reference (with Inline Comments)
``` yaml
# eksctl schema version
apiVersion: eksctl.io/v1alpha5

# Declares that this file defines an EKS cluster config
kind: ClusterConfig

metadata:
  # Cluster name (dynamic based on CLUSTER_NAME env var)
  name: ${CLUSTER_NAME}-uswt2-eks-cluster

  # AWS region where the cluster will be created
  region: us-west-2

  # EKS Kubernetes version
  version: "1.32"

vpc:
  # Existing VPC ID where the cluster will be deployed
  id: vpc-08a80834636025c17

  # Subnet mappings for EKS control plane + nodes
  subnets:
    private: # Private subnets used for worker nodes
      us-west-2a:
        # Existing private subnet ID in AZ 2a
        id: subnet-0b3a6a1bb2cd49607
      us-west-2b:
        # Existing private subnet ID in AZ 2b
        id: subnet-0ec2230b5b40f39c4

    public: # Public subnets (mainly for ALB load balancers)
      us-west-2a:
        # Existing public subnet ID in AZ 2a
        id: subnet-0b3201ea206ee015d
      us-west-2b:
        # Existing public subnet ID in AZ 2b
        id: subnet-0c81996babae4b7b2

  clusterEndpoints:
    # Allow API access from the public internet
    publicAccess: true

    # Allow API access from inside the VPC
    privateAccess: true

managedNodeGroups:
  # Each element defines one node group
  - name: ng-${CLUSTER_NAME}-eks-cluster  # Node group name

    # EC2 instance type for worker nodes
    instanceType: t3.medium

    # Initial number of nodes in the node group
    desiredCapacity: 2

    # Minimum autoscaling limit
    minSize: 1

    # Maximum autoscaling limit
    maxSize: 3

    # Specify AMI family (Amazon Linux 2023 optimized for EKS)
    amiFamily: AmazonLinux2023

    # Custom AMI ID if needed (overrides default AL2023 image)
    ami: ami-063a81f8743fece51

    # Ensure nodes are created ONLY in private subnets
    privateNetworking: true

    ssh:
      # Allow SSH access to nodes
      allow: true

      # EC2 keypair to attach for SSH logins
      publicKeyName: ${CLUSTER_NAME}-sshkey

    securityGroups:
      # Attach existing security groups to the node group
      attachIDs:
        - sg-03afdf513c0c04772

    labels:
      # Kubernetes labels applied to the node group
      role: app

    tags:
      # AWS tags applied to EC2 instances
      Name: ng-${CLUSTER_NAME}-eks-cluster

cloudWatch:
  clusterLogging:
    # Enable specific EKS control plane logs
    enableTypes:
      - api             # Kubernetes API server logs
      - audit           # Audit logs
      - authenticator   # Authenticator component logs

iam:
  # Enable IAM OIDC provider for IRSA (Pod -> IAM role access)
  withOIDC: true

autoModeConfig:
    # Enable EKS Auto-Mode for managed compute/storage/networking
    enabled: true

```
