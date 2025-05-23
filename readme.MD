# Ingress controller

#### PERMISSIONS ##############
## EKS is a different cluster and ALB is different service, EKS Cluster need to create and ALB Hence using this service.
REGION_CODE=us-east-1
CLUSTER_NAME=expense
ACC_ID=590184056608

Step1: OIDC Provider
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve

Step2: IAM Policy
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.12.0/docs/install/iam_policy.json

Step3: Create IAM ROLE
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

Step4: Provide access to EKS through IAM Policy
eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve

#### INSTALL DRIVERS ##############

step1: Add the EKS chart repo to Helm
helm repo add eks https://aws.github.io/eks-charts

step2: Install AWS Load balancer controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller