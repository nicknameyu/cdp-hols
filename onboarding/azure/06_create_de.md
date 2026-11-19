# Create Data Engineer

## Create public Data Engineer Cluster

### Prepare variables
```
SERVICE_MI=$(terraform output -json | jq -r .data_engineer_mi_ids.value.cluster1_service.id)
CLUSTER_MI=$(terraform output -json | jq -r .data_engineer_mi_ids.value.cluster1_cluster.id)
DE_NAME=dyu-az-de-poc
ENVNAME=dyu-az-poc-env
SUBNET=pub_subnet_3
VM_TYPE=Standard_E16s_v5
```

### Create Data Engineering
```
cdp de enable-service \
 --name $DE_NAME \
 --env $ENVNAME \
 --instance-type $VM_TYPE \
 --minimum-instances 0 \
 --maximum-instances 50 \
 --enable-public-endpoint \
 --no-enable-private-network \
 --subnets $SUBNET \
 --azure-service-managed-identity $SERVICE_MI \
 --azure-virtual-cluster-managed-identities $CLUSTER_MI 
```

## Create public Data Engineer Cluster