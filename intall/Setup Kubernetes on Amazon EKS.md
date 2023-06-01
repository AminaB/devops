Setup Kubernetes on Amazon EKS

You can follow same procedure in the official AWS document Getting started with Amazon EKS – eksctl
Pre-requisites:

    an EC2 Instance
    Install AWSCLI latest verison

    Setup kubectl
    a. Download kubectl version 1.21
    b. Grant execution permissions to kubectl executable
    c. Move kubectl onto /usr/local/bin
    d. Test that your kubectl installation was successful

    curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    mv ./kubectl /usr/local/bin 
    kubectl version --short --client

Setup eksctl
a. Download and extract the latest release
b. Move the extracted binary to /usr/local/bin
c. Test that your eksclt installation was successful

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

Create an IAM Role and attache it to EC2 instance
Note: create IAM user with programmatic access if your bootstrap system is outside of AWS
IAM user should have access to
IAM
EC2
CloudFormation
Note: Check eksctl documentaiton for Minimum IAM policies

Create your cluster and nodes

eksctl create cluster --name cluster-name  \
--region region-name \
--node-type instance-type \
--nodes-min 2 \
--nodes-max 2 \
--zones <AZ-1>,<AZ-2>

example:
eksctl create cluster --name valaxy-cluster \
--region ap-south-1 \
--node-type t2.small \

To delete the EKS clsuter

eksctl delete cluster valaxy --region ap-south-1

Validate your cluster using by creating by checking nodes and by creating a pod

kubectl get nodes
kubectl run tomcat --image=tomcat

Deploying Nginx pods on Kubernetes

Deploying Nginx Container

kubectl create deployment  demo-nginx --image=nginx --replicas=2 --port=80
# kubectl deployment regapp --image=valaxy/regapp --replicas=2 --port=8080
kubectl get all
kubectl get pod

Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.

kubectl expose deployment demo-nginx --port=80 --type=LoadBalancer
# kubectl expose deployment regapp --port=8080 --type=LoadBalancer
kubectl get services -o wide

