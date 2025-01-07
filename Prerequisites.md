# Stan's Robot - 3-Tier Application on EKS 🤖💻
---
![website-full-future](https://github.com/user-attachments/assets/e8b0f78b-86fb-49a9-8f25-81fee00c3e22)

---
Stan's Robot is a 3-tier application deployed on Amazon EKS (Elastic Kubernetes Service). This project demonstrates how to create and configure an EKS cluster, deploy a 3-tier architecture, and integrate various AWS services like the AWS Load Balancer Controller and EBS CSI plugin for persistent storage.

## Project Overview 🌍🚀

In this demo, we will deploy a 3-tier application using Amazon EKS. The setup includes:

- **EKS Cluster creation**
- **AWS Load Balancer Controller installation**
- **EBS CSI Plugin configuration for persistent storage**

This project aims to help you understand and practice deploying modern, scalable applications in the cloud using AWS tools and services.

### Project Deployment Flow:

**Architecture Diagram:**

(![EKS-Architecture-Stan's-Robot-Shop](https://github.com/user-attachments/assets/2c542b0b-9a7b-453b-bbe9-a948ee3b5fef)


## Prerequisites 📋

Before you begin, make sure you have the following tools installed:

- **kubectl**: A command-line tool for interacting with Kubernetes clusters.  
  [Installation guide: Install kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)

- **eksctl**: A command-line tool for managing EKS clusters.  
  [Installation guide: Install eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

- **AWS CLI**: AWS CLI for interacting with AWS services.  
  [Installation guide: Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

After installing AWS CLI, configure it by running:

```bash
aws configure
```
## Setup Instructions 🚀

### 1. Create EKS Cluster 🌐
To create the EKS cluster using eksctl, run:

```bash
eksctl create cluster --name demo-cluster-three-tier-1 --region ap-south-1
```

To delete the EKS cluster:
```bash
eksctl delete cluster --name demo-cluster-three-tier-1 --region ap-south-1
```
### 2. Configure IAM OIDC Provider 🔑
Set up the IAM OIDC provider to enable Kubernetes workloads to securely interact with AWS services.

Set your cluster name:
```bash
export cluster_name=<CLUSTER-NAME>
```

Retrieve the OIDC issuer ID:
```bash
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
```

Check if the OIDC provider is already configured:
```bash
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```
If not, associate the OIDC provider:
```bash
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

### 3. Install ALB Add-On (AWS Load Balancer Controller) 🛠️

Download the IAM policy for the ALB controller:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

- Create the IAM policy:
```bash
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```
- Create an IAM role for the ALB controller:
```bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
- Add the Helm repo and install the ALB controller:
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```

```helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
- Verify the deployment:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
### 4. Configure EBS CSI Plugin for Persistent Storage 📦
- Create the IAM role for EBS CSI:
```bash
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster <YOUR-CLUSTER-NAME> \
  --role-name AmazonEKS_EBS_CSI_DriverRole \
  --role-only \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve
```
- Install the EBS CSI driver as an add-on:
```bash
eksctl create addon --name aws-ebs-csi-driver --cluster <YOUR-CLUSTER-NAME> --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force
```

## References 🔗
- AWS EKS Documentation
- AWS CLI Documentation
- eksctl Documentation
- AWS Load Balancer Controller Documentation
- EBS Persistent Storage Guide

## Note:
Replace placeholders like <your-cluster-name>, <region>, and <AWS-ACCOUNT-ID> with your actual values.

This project provides a foundation to deploy scalable applications on AWS using Kubernetes and various AWS tools.
