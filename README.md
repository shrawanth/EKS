
### Prerequisites
- Download and Install AWS Cli - Please Refer [this](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) link.
- Setup and configure AWS CLI using the `aws configure` command.
- Install and configure eksctl using the steps mentioned [here](https://eksctl.io/installation/).
- Install and configure kubectl as mentioned [here](https://kubernetes.io/docs/tasks/tools/).

# 🛠️  Installation & Configurations
## 📦 Step 1: Create EKS Cluster using fargate
```bash
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```
Update ./kube/config file
```bash
aws eks update-kubeconfig --name demo-cluster --region us-east-1
```
## Step 2: Create Fargate profile
```bash
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```
## Step 3: Deploy the deployment, service and Ingress
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
## Step 4: Commands to configure IAM OIDC provider
```bash
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
*********
## step 5: Create IAM Policy & IAM Role
Create IAM Policy
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
Create IAM Role
```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
- Make sure you replace `<your-aws-account-id>` with aws account ID.
## Step 6: Deploy ALB controller
Add helm repo
```bash
helm repo add eks https://aws.github.io/eks-charts
```
Update the repo
```bash
helm repo update eks
```
Install
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
- Make sure you replace `<your-vpc-id>`, `<region>`, `<your-cluster-name>` respectively using aws ui.
Verify that the deployments are running.
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
## Step 7: 
```bash
kubectl get ingress -n game-2048
```
