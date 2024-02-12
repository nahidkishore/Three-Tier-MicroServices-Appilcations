## Commands lines for this projects:
```
git clone https://github.com/nahidkishore/Three-Tier-MicroServices-Appilcations.git
```
### aws cli install 
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
 ```   
### kubectl install 
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
### eksctl installtion 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
### helm installtions 
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
## go to our main application folder 
cd Three-Tier-MicroServices-Appilcations/
ls
### create eks cluster 
```
eksctl create cluster --name three-tier-cluster --region us-east-1

export cluster_name=three-tier-cluster

oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)

aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
### alb-configuration or setup
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

aws iam create-policy     --policy-name AWSLoadBalancerControllerIAMPolicy     --policy-document file://iam_policy.json

eksctl create iamserviceaccount   --cluster=three-tier-cluster-1   --namespace=kube-system   --name=aws-load-balancer-controller   --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::777544701073:policy/AWSLoadBalancerControllerIAMPolicy   --approve
```
 ### now go 
```
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=three-tier-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-0145c9c9f06133263
```
##### Confirm that the deployments are running successfully by checking the AWS Load Balancer Controller deployment in the kube-system namespace.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
## EBS CSI Plugin configuration
```
eksctl create iamserviceaccount     --name ebs-csi-controller-sa     --namespace kube-system     --cluster three-tier-cluster     --role-name AmazonEKS_EBS_CSI_DriverRole     --role-only     --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy     --approve

eksctl create addon --name aws-ebs-csi-driver --cluster three-tier-cluster-1 --service-account-role-arn arn:aws:iam::777544701073:role/AmazonEKS_EBS_CSI_DriverRole --force
```
#### now go to our eks helm folder, where we have already setup our projects deployment and service yaml file

cd EKS/
ls
cd helm/
ls
### create namsepsace 
```
kubectl create ns robot-shop
```
##### Executing this command deploys the entire Robot Shop project, utilizing the Helm chart to manage each component seamlessly.
```
helm install robot-shop --namespace robot-shop .
```
### check pods and svc 
```
kubectl get pods -n robot-shop
kubectl get svc -n robot-shop
```
##### Two options exist for exposing the application externally: through a Load Balancer or via an Ingress Controller. Since an Ingress Controller is already set up, we opt for this method.
```
ls
cat ingress.yaml
```
### This command applies the Ingress resource configuration, allowing external access to the Robot Shop application.
```
kubectl apply -f ingress.yaml
kubectl get ingress -n robot-shop

```