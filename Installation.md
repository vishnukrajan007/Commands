🔹 Step 1: Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
aws --version


Configure:

aws configure

🔹 Step 2: Install Terraform
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl

curl -fsSL https://apt.releases.hashicorp.com/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install terraform -y

terraform -version

🔹 Step 3: Deploy Infrastructure
git clone https://github.com/jaiswaladi246/Mega-Project-Terraform.git
cd Mega-Project-Terraform

terraform init
terraform apply --auto-approve

🔹 Step 4: Update Kubeconfig
aws eks --region ap-south-1 update-kubeconfig --name devopsshack-cluster

🔹 Step 5: Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client

🔹 Step 6: Install eksctl
curl --silent --location \
"https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
| tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
eksctl version

🔹 Step 7: Create EKS Cluster
eksctl create cluster \
--name=EKS-1 \
--region=ap-south-1 \
--zones=ap-south-1a,ap-south-1b \
--without-nodegroup

🔹 Step 8: Enable OIDC
eksctl utils associate-iam-oidc-provider \
--region ap-south-1 \
--cluster EKS-1 \
--approve

🔹 Step 9: Create Nodegroup
eksctl create nodegroup \
--cluster=EKS-1 \
--region=ap-south-1 \
--name=node2 \
--node-type=t3.medium \
--nodes=3 \
--nodes-min=2 \
--nodes-max=4 \
--node-volume-size=20 \
--ssh-access \
--ssh-public-key=DevOps \
--managed

🔹 Step 10: Install EBS CSI Driver
eksctl create iamserviceaccount \
--region ap-south-1 \
--name ebs-csi-controller-sa \
--namespace kube-system \
--cluster devopsshack-cluster \
--attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
--approve

kubectl apply -k \
"github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.11"

🔹 Step 11: Install NGINX Ingress
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

🔹 Step 12: Install Cert-Manager
kubectl apply -f \
https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml

🔹 Step 13: Install ArgoCD
kubectl create namespace argocd

kubectl apply -n argocd -f \
https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Get Password:

kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d; echo

🔹 Step 14: Run Nexus
docker run -d --name nexus -p 8081:8081 sonatype/nexus3:latest

🔹 Step 15: Run SonarQube
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
