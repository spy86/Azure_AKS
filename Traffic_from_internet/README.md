# Traffic_from_internet

## Requirements:

* AKS Cluster
* Basic External LoadBalancer

#### Create Resource group

```bash
az group create --location westeurope --resource-group demo-weu-rg
```

#### Create Service Principal

```bash
az ad sp create-for-rbac --skip-assignment -n "spn-aks"
```

#### Create Azure Kubernetes Service

> **_NOTE:_** We need to change --subscription, --service-principal, --client-secret

```bash
az aks create \
  --location westeurope \
  --subscription 00000000-0000-0000-0000-000000000000 \
  --resource-group demo-weu-rg \
  --name 1386e4a2-8f22-weu-aks \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "00000000-0000-0000-0000-000000000000" \
  --client-secret "00000000-0000-0000-0000-000000000000" \
  --network-plugin kubenet \
  --load-balancer-sku standard \
  --outbound-type loadBalancer \
  --node-vm-size Standard_B2s \
  --node-count 1 \
  --tags 'ENV=Demo' 'OWNER=Corporation Inc.'
```

> **_NOTE:_** Now we have to wait a while until our cluster is created

#### Get kubeconfig
```bash
az aks get-credentials \
  --resource-group demo-weu-rg \
  --name 1386e4a2-8f22-weu-aks \
  --admin
```


#### Create an ingress controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx --version 4.1.3 --namespace ingress-nginx --create-namespace --set controller.replicaCount=1 --set controller.nodeSelector."kubernetes\.io/os"=linux --set controller.admissionWebhooks.patch.nodeSelector."kubernetes\.io/os"=linux --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux
```
#### Check the load balancer service

```bash
kubectl get services --namespace ingress-nginx -o wide -w ingress-nginx-controller
```

#### Deploy Application

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

## Testing

#### Open URL from webbrowser

1. http://IP-FROM-OUR-INGRESS/
2. http://IP-FROM-OUR-INGRESS/hello-world-two
3. http://IP-FROM-OUR-INGRESS/static


#### CleanUP
```bash
az group delete -n demo-weu-rg --yes --no-wait
```