# Amazon EBS CSI driver

Lỗi này xảy ra khi CSI Driver không được thực thi. Ảnh hưởng tới việc tạo Persistance Volume

Cách giải quyết : [Amazon EBS CSI driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html) => cái này cũng chính là NFS trong kubernetes

- Read debug Log [kubectk cheatsheet](https://kubernetes.io/vi/docs/reference/kubectl/cheatsheet/3)
# Create EKS Cluster & Node Groups

## Step-00: Introduction
- Understand about EKS Core Objects
  - Control Plane
  - Worker Nodes & Node Groups
  - Fargate Profiles
  - VPC
- Create EKS Cluster
- Associate EKS Cluster to IAM OIDC Provider
- Create EKS Node Groups
- Verify Cluster, Node Groups, EC2 Instances, IAM Policies and Node Groups

## Step-01: Create EKS Cluster using eksctl
- It will take 15 to 20 minutes to create the Cluster Control Plane 
```bash
# Create Cluster
eksctl create cluster --name=sgfintech \
                      --region=ap-southeast-1 \
                      --zones=ap-southeast-1a,ap-southeast-1b \
                      --without-nodegroup 

# Get List of clusters
eksctl get clusters                  
```


## Step-02: Create & Associate IAM OIDC Provider for our EKS Cluster
- To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create &  associate OIDC identity provider.
- To do so using `eksctl` we can use the  below command. 
- Use latest eksctl version (as on today the latest version is `0.21.0`)
```bash                   
# Template
eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluter-name> \
    --approve

# Replace with region & cluster name
eksctl utils associate-iam-oidc-provider \
    --region ap-southeast-1 \
    --cluster sgfintech-dev \
    --approve
```

## Step-03: Create EC2 Keypair
- Create a new EC2 Keypair with name as `matkinhig-services-keypair`
- This keypair we will use it when creating the EKS NodeGroup.
- This will help us to login to the EKS Worker Nodes using Terminal.

## Step-04: Create Node Group with additional Add-Ons in Public Subnets
- These add-ons will create the respective IAM policies for us automatically within our Node Group role.
 ```bash
# Create Public Node Group   
eksctl create nodegroup --cluster=sgfintech-dev \
                        --region=ap-southeast-1 \
                        --name=sgfintech-nodegroup-public \
                        --node-type=t2.micro \
                        --nodes=1 \
                        --nodes-min=1 \
                        --nodes-max=2 \
                        --node-volume-size=100 \
                        --ssh-access \
                        --ssh-public-key=matkinhig-services-keypair \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access
```
If edit amount of nodes :

```bash
eksctl scale nodegroup --cluster=sgfintech-bnpl \
                    --nodes=1 \
                    --name=sgfintech--nodegroup-public1 \
                    --nodes-min=1
```

If delete nodegroup :

```bash
eksctl delete nodegroup --cluster=sgfintech-dev --name=sgfintech-nodegroup-public1 --disable-eviction
```

## Step-05: Verify Cluster & Nodes

### Verify NodeGroup subnets to confirm EC2 Instances are in Public Subnet
- Verify the node group subnet to ensure it created in public subnets
  - Go to Services -> EKS -> sgfintech-bnpl -> sgfintech--nodegroup-public1
  - Click on Associated subnet in **Details** tab
  - Click on **Route Table** Tab.
  - We should see that internet route via Internet Gateway (0.0.0.0/0 -> igw-xxxxxxxx)

### Verify Cluster, NodeGroup in EKS Management Console
- Go to Services -> Elastic Kubernetes Service -> sgfintech-bnpl

### List Worker Nodes
```bash
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=sgfintech-dev

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# Our kubectl context should be automatically changed to new cluster
kubectl config view --minify
```

### Verify Worker Node IAM Role and list of Policies
- Go to Services -> EC2 -> Worker Nodes
- Click on **IAM Role associated to EC2 Worker Nodes**

### Verify Security Group Associated to Worker Nodes
- Go to Services -> EC2 -> Worker Nodes
- Click on **Security Group** associated to EC2 Instance which contains `remote` in the name.

### Verify CloudFormation Stacks
- Verify Control Plane Stack & Events
- Verify NodeGroup Stack & Events

### Login to Worker Node using Keypai kube-demo
- Login to worker node
```
# For MAC or Linux
ssh -i matkinhig-services-keypair.pem ec2-user@<Public-IP-of-Worker-Node>

```

## Step-06: Update Worker Nodes Security Group to allow all traffic
- We need to allow `All Traffic` on worker node security group

## Additional References
- https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
- https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html

# Create Wordpress Application with MYSQL

This app using kutomization manager package and application deploy to Cluster k8s

```kubectl
kubectl apply -k ./
```

If want delete This App :

```kubectl
kubectl delete -k ./
```

# Create Wordpress Application with Helm


Update repo Helm

```
helm repo add stable https://charts.helm.sh/stable --force-update
```

Install App : 

```
helm install my-release \                                        
  --set wordpressUsername=admin \
  --set wordpressPassword=password \
  --set mariadb.mariadbRootPassword=secretpassword \
    stable/wordpress
```


if want use bitnami , need Add repo Bitnami

```
 helm repo add bitnami https://charts.bitnami.com/bitnami --force-update
```

if you want delete this app 

```
helm uninstall my-release
```





