<!-- README.md:
Updated Architecture Diagram (Draw.io or other tool).
Brief application explanation.
Deployment instructions.
Links Table: Repository links and Docker Hub image links for all services. -->

# CST8915 Final Project: Best Buy on Kubernetes

**Student Name**: Anoop Sidhu
**Student ID**: 040984994
**Course**: CST8915 Full-stack Cloud-native Development
**Semester**: Winter 2026

---
## Demo Video

🎥 [Watch Demo Video]()

---

## Architecture Diagram

---
## Technical Explanation of Task 2

- Using Kubernetes PersistentVolumeClaims to add persistence to Mongo.
- Volume mounted on `/data/db`
- Increased mongodb replicas to 3 for increased durability.
- Added storage block, which persists even if pod is deleted.
- Verified by querying for the same `ObjectId` between deployments.
- Used same pattern on RabbitMQ, persisting `/var/lib/rabbitmq`, unfortunately could not achieve persistence.

---
## Deploy to Azure Kubernetes Service (AKS)

These steps deploy this project to a new AKS cluster using the manifests in this repository.

### 1) Prerequisites

- Azure subscription with permission to create resource groups and AKS clusters
- `az` (Azure CLI), `kubectl`, and `git` installed locally
- Docker images must be available in Docker Hub (this repo currently references `ansid0cker/*` images)

### 2) Sign in and set Azure subscription

```bash
az login
az account set --subscription "<YOUR_SUBSCRIPTION_NAME_OR_ID>"
az account show --output table
```

### 3) Create resource group and AKS cluster

```bash
# Set your values
RESOURCE_GROUP="rg-bestbuy-aks"
LOCATION="canadacentral"
AKS_NAME="aks-bestbuy"

az group create --name "$RESOURCE_GROUP" --location "$LOCATION"

az aks create \
	--resource-group "$RESOURCE_GROUP" \
	--name "$AKS_NAME" \
	--node-count 2 \
	--node-vm-size Standard_B2s \
	--generate-ssh-keys
```

### 4) Connect `kubectl` to your AKS cluster

```bash
az aks get-credentials --resource-group "$RESOURCE_GROUP" --name "$AKS_NAME" --overwrite-existing
kubectl get nodes
```

### 5) Create a namespace for the app

```bash
kubectl create namespace bestbuy
kubectl config set-context --current --namespace=bestbuy
```

### 6) Validate manifests before deploy

```bash
kubectl apply --dry-run=client -f "Deployment Files/" -R
```

If validation fails for a manifest, fix that file and re-run the dry run before continuing.

### 7) Deploy all manifests

```bash
kubectl apply -f "Deployment Files/config/"
kubectl apply -f "Deployment Files/statefulsets/"
kubectl apply -f "Deployment Files/deployments/"
kubectl apply -f "Deployment Files/services/"
```

### 8) Verify workloads

```bash
kubectl get pods -o wide
kubectl get deployments
kubectl get statefulsets
kubectl describe pod <POD_NAME>
kubectl logs <POD_NAME>
```

### 9) Access the application

The `Deployment Files/services/` directory includes `LoadBalancer` services for the store-front and store-admin deployments. Monitor their external IPs:

```bash
kubectl get svc -w
```

Wait for `EXTERNAL-IP` to be assigned, then navigate to those IPs in your browser to access the frontend and admin panel.

### 10) Clean up Azure resources

```bash
az group delete --name "$RESOURCE_GROUP" --yes --no-wait
```

---
Links

