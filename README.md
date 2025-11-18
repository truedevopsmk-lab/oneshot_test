apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${CLUSTER_NAME}-uswt2-eks-cluster
  region: us-west-2
  version: "1.32"

vpc:
  id: vpc-08a80834636025c17
  subnets:
    private:
      us-west-2a:
        id: subnet-0b3a6a1bb2cd49607
      us-west-2b:
        id: subnet-0ec2230b5b40f39c4
    public:
      us-west-2a:
        id: subnet-0b3201ea206ee015d
      us-west-2b:
        id: subnet-0c81996babae4b7b2

  clusterEndpoints:
    publicAccess: true
    privateAccess: true

managedNodeGroups:
  - name: ng-${CLUSTER_NAME}-eks-cluster
    instanceType: t3.medium
    desiredCapacity: 2
    minSize: 1
    maxSize: 3

    amiFamily: AmazonLinux2023
    ami: ami-063a81f8743fece51

    privateNetworking: true

    ssh:
      allow: true
      publicKeyName: ${CLUSTER_NAME}-sshkey

    securityGroups:
      attachIDs:
        - sg-03afdf513c0c04772

    labels:
      role: app

    tags:
      Name: ng-${CLUSTER_NAME}-eks-cluster

cloudWatch:
  clusterLogging:
    enableTypes:
      - api
      - audit
      - authenticator

iam:
  withOIDC: true

autoModeConfig:
    enabled: true
