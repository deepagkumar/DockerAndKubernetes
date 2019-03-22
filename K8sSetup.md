# Kubernetes setup

## K8s in AWS with kops
A t2 micro instance with Ubuntu linux AMI. This will be used for managing the cluster.
An IAM user with the following permissions
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
AmazonVPCFullAccess
IAMFullAccess
### Install latest release of kubectl on Linux
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl

#### Install "kops" on Linux
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
kops version

#### Install and configure the AWS CLI on Linux
sudo apt-get install awscli
aws configure

<aws_key>
<aws-secret>
ap-southeast-1 (S3 does not work as of now with ap-south-1)

#### Create and then list a new S3 bucket 
aws s3 mb s3://cluster-1.aws-ha-k8s.bigdecisionsdemo.com
aws s3 ls | grep k8s

export KOPS_STATE_STORE=s3://cluster-1.aws-ha-k8s.bigdecisionsdemo.com (Add this to .bashrc/.profile)

#### Create cluster
kops create cluster \
--cloud=aws --zones="ap-south-1a,ap-south-1b" \
--dns-zone=bigdecisionsdemo.com \
--name=cluster-1.aws-ha-k8s.bigdecisionsdemo.com \
--vpc=<my-vpc> \
--node-size=m4.2xlarge \
--node-count=3 \
--node-security-groups=<mshost-security-group> --yes

kops create cluster \
--cloud=aws --zones="ap-south-1a,ap-south-1b" \
--dns-zone=bigdecisionsdemo.com \
--name=cluster-1.aws-ha-k8s.bigdecisionsdemo.com \
--vpc=vpc-02dcfb90fac1b76f1 \
--node-size=m4.2xlarge \
--node-count=3 \
--node-security-groups=sg-06e219721d2e56c08

## Useful Commands:
* list clusters with: kops get cluster
* edit this cluster with: kops edit cluster cluster-1.aws-ha-k8s.bigdecisionsdemo.com
* edit your node instance group: kops edit ig --name=cluster-1.aws-ha-k8s.bigdecisionsdemo.com nodes
* edit your master instance group: kops edit ig --name=cluster-1.aws-ha-k8s.bigdecisionsdemo.com master-ap-south-1a

kops update cluster cluster-1.aws-ha-k8s.bigdecisionsdemo.com --yes

kops delete cluster --name cluster-1.aws-ha-k8s.bigdecisionsdemo.com --yes

To validate cluster: kops validate cluster
To list nodes: kubectl get nodes --show-labels

kubectl create configmap bigdecisions-conf-infrastructure --from-file=/opt/bigdecisions/deployment/conf/infrastructure.properties
kubectl create configmap bigdecisions-conf-processflowengine --from-file=/opt/bigdecisions/deployment/conf/processflowengine.properties

kubectl create secret docker-registry nexusdockercred --docker-server=https://nexus.bigdecisionsdemo.com:8445 --docker-username="user" --docker-password=pwd --docker-email=deploy@example.com

### Kubernetes dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl cluster-info
Dashboard url: https://api.cluster-1.aws-ha-k8s.bigdecisionsdemo.com/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
userid: admin
password: Execute this command to get password(kops get secrets kube --type secret -oplaintext) or (kubectl config view)
<pwd>
Token: Execute this command to get token(kops get secrets admin --type secret -oplaintext)
<token>

Logs from the pods can be viewed from dashboard. Select pods->the required pod and click on logs

kubectl get pod -o=custom-columns=NODE:.spec.nodeName,NAME:.metadata.name

# Kubernetes in Azure and On-prem using kubeadm
## Repeat the below in all nodes of the cluster
apt-get update && apt-get install -y apt-transport-https

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update

apt-get install -y docker.io kubeadm kubectl kubelet kubernetes-cni

systemctl enable docker.service

## Do this only on the master nodes
kubeadm init

## Note the cluster joining command from kubeadm init command
## exit to regular user

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes

kubectl get pods --all-namespaces

## Setup networking
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubectl get nodes

kubectl get pods --all-namespaces

## To join the worker nodes. Execute the join command from the kubeadm init
kubeadm join 10.2.1.1:6443 --token <token> --discovery-token-ca-cert-hash sha256:121212121111121111212
