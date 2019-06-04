## Instructions for Installing on ACI

Create a Resource Group 
```shell
az group create --name OSSDevopsHackathon --location eastus
```
Deploy the YAML for the Jenkins Container

```shell
az container create --resource-group OSSDevopsHackathon -f osshackjenkins.yaml
```

## Instructions for Installation on Kubernetes

For this tutorial, I will use OSSDevopsHackathon as my resource group name, OSSDevopsNetwork and OSSDevopsSubnet01 as my virtual network and subnet names respectively. I will name the AKS cluster and the service principal OSSDevopsHackathonCluster and OSSDevopsSP03 respectively.

## Create Resource Group
Create a Resource Group 
```shell
az group create --name OSSDevopsHackathon --location eastus
```

## Create Virtual Network

Create a virtual network and subnet
```shell
az network vnet create -g OSSDevopsHackathon -n OSSDevopsNetwork --address-prefix 10.59.0.0/16 --subnet-name OSSDevopsSubnet01 --subnet-prefix 10.59.0.0/18
```

## List the Subnets 

Retrieve the Subnet Ids from the VNet you created

```shell
az network vnet subnet list -g OSSDevopsHackathon --vnet-name OSSDevopsNetwork --query [].id
```

# Create Service Principal

Create a service principal with contributor access to your subscription

```shell
az ad sp create-for-rbac --role="Contributor" --name="OSSDevopsSP03"

```
# Create Kubernetes Cluster

Create a K8S Cluster using the information from the resources generated earlier

```shell
az aks create --kubernetes-version 1.13.5 --resource-group OSSDevopsHackathon --name OSSDevopsHackathonCluster --location eastus --node-count 3 --max-pods 250 --node-vm-size Standard_D4s_v3 --network-plugin azure --vnet-subnet-id "/subscriptions/96d61f02-2fcf-4a37-bb6e-cc2c2db9f67d/resourceGroups/OSSDevopsHackathon/providers/Microsoft.Network/virtualNetworks/OSSDevopsNetwork/subnets/OSSDevopsSubnet01" --docker-bridge-address 172.17.0.1/16 --dns-service-ip 10.58.0.10 --service-cidr 10.58.0.0/16 --generate-ssh-keys --service-principal 9d29e06e-af69-4af6-a6ae-0e3d9ff66081 --client-secret 69bc73dd-0ab3-470d-979e-d7149949fe91 --tags project=OSSDevops --disable-rbac
```

# If you do not have the Kubectl CLI program, run the following command:

Install kubectl (Kube Control) if you don't already have it locally

```shell
az aks install-cli
```

# Get the AKS Credentials

Retrieve the certificate data for authenticating with the remote cluster

```shell
az aks get-credentials --resource-group OSSDevopsHackathon --name OSSDevopsHackathonCluster
az aks get-credentials -g OSSDevopsHackathon -n OSSDevopsHackathonCluster
```

# Show the dashboard for a Kubernetes cluster in a web browser

If you need to see the dashboard, please use the command below to set up the tunnel:

```shell
az aks browse --name OSSDevopsHackathonCluster --resource-group OSSDevopsHackathon
```

# Deploy the Jenkins App and K8S Loadbalancer

The commands below will deploy the jenkins:lts image to the kubernetes cluster and set up a load balancer to access it.

```shell
kubectl apply -f jenkins-deployment.yml
kubectl apply -f jenkins-services.yml 
```

## Accessing the Jenkins App

Once the app is up and running you should be able to get the service IPs by running the following commands:

Then you can access it on port 80 for the jenkins service and on port 8019 on the jenkins-special service

```shell
kubectl get services
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
jenkins           LoadBalancer   10.58.48.21     40.87.121.55    80:30725/TCP     19m
jenkins-jnlp      ClusterIP      10.58.161.130   <none>          50000/TCP        19m
jenkins-special   LoadBalancer   10.58.69.202    40.121.34.56    8019:31191/TCP   12m
kubernetes        ClusterIP      10.58.0.1       <none>          443/TCP          76m

```

## Setting up Prometheus on the K8S cluster using the Helm

Setting up and Installing Helm Locally

```shell

# Get Helm installation script
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get -o get_helm.sh

# Make the script executable locally
chmod 700 get_helm.sh

# Execute the script
./get_helm.sh

# Initialize Helm
helm init

```

Install Prometheus Under the Release Name oss-hack-release

```shell
helm install --name oss-hack-release stable/prometheus
```

If you want to uninstall the Prometheus chart, please execute the following command

```shell

helm delete oss-hack-release
```
For additional information, please refer to the Prometheus repo
https://github.com/helm/charts/tree/master/stable/prometheus

