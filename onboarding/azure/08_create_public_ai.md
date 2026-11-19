# Create Public Cloudera AI

## Create AI Workbench
** Note: ** Housekeeping
CAI Workbench rely on an empty NFS folder. If a provisioning fails, always recreate the NFS folder.
```
terraform apply -replace='module.env-prerequisite.azurerm_storage_share.nfs[0]'
```

** Step 1: ** Prepare variables
```
NFS=$(terraform output -json | jq -r '.storage_locations.value["nfs-file-share"].nfs_file_share')
ENVNAME=dyu-az-poc-env
WBNAME=dyu-az-ai-wb
SUBNET=pub_subnet_7
PROVISION_REQUEST=$(jq -n \
  --arg subnet "$SUBNET" \
'{
    "environmentName": "dyu-az-poc-env",
    "instanceGroups": [
      {
        "name": "liftie-infra",
        "instanceType": "Standard_DS4_v2",
        "instanceCount": 2,
        "rootVolume": { "size": 256 },
        "autoscaling": {
          "enabled": true,
          "minInstances": 2,
          "maxInstances": 3
        }
      },
      {
        "name": "mlinfra",
        "instanceType": "Standard_DS4_v2",
        "instanceCount": 2,
        "rootVolume": { "size": 256 },
        "autoscaling": {
          "enabled": true,
          "minInstances": 2,
          "maxInstances": 3
        }
      },
      {
        "name": "mlcpu0",
        "instanceType": "Standard_DS5_v2",
        "instanceCount": 1,
        "rootVolume": { "size": 256 },
        "autoscaling": {
          "enabled": true,
          "minInstances": 0,
          "maxInstances": 10
        }
      }
    ],
    "network": {
      "topology": {
        "subnets": [$subnet]
      }
    }
  }')
```

** Step 2: ** Create Workbench
```
cdp ml create-workspace \
--environment-name $ENVNAME \
--workspace-name $WBNAME \
--use-public-load-balancer \
--no-private-cluster \
--enable-monitoring \
--enable-governance \
--existing-nfs $NFS \
--nfs-version 4.1 \
--subnets-for-load-balancers $SUBNET \
--provision-k8s-request $PROVISION_REQUEST
```


## Create AI Registry
** Step 1: ** Prepare variable
```
ENVNAME=dyu-az-poc-env
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
ENVCRN=$(echo $ENVDETAILS |jq .environment.crn|tr -d '"')
SUBNET=pub_subnet_6
```
** Step 2: ** Run Command
```
cdp ml create-model-registry \
  --environment-crn $ENVCRN \ 
  --environment-name $ENVNAME \  
  --no-private-cluster \
  --use-public-load-balancer \
  --provision-k8s-request '{"network":{"topology":{"subnets":["'"${SUBNET}"'"]}}}'
```
## Create AI Inference Server
** Step 1: ** Initiate Environment Default Compute Cluster
1. Prepare variable
```
ENVNAME=dyu-az-poc-env
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
INFNAME=dyu-inf-svc
ENVCRN=$(echo $ENVDETAILS |jq .environment.crn|tr -d '"')
SUBNET=pub_subnet_5
MY_IP="<My internet IP>/32"
```
2. Run Command
```
cdp environments initialize-azure-compute-cluster \
--environment-name $ENVNAME \
--compute-cluster-configuration privateCluster=false,workerNodeSubnets=${SUBNET},kubeApiAuthorizedIpRanges=${MY_IP}
```

** Step 2: ** Create CAI Inference Compute Cluster
```
CLUSTER_NAME=dyu-az-inf-cluster
cdp compute create-cluster \
--environment $ENVNAME \
--name $CLUSTER_NAME \
--network subnets=$SUBNET
```

** Step 3: ** Deploy CAI Inference Service
```
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
INFNAME=dyu-inf-svc
ENVCRN=$(echo $ENVDETAILS |jq .environment.crn|tr -d '"')
CLUSTERCRN=$(cdp compute list-clusters | jq -r ".clusters[] | select(.envName == \"$ENVNAME\" and .clusterName == \"$CLUSTER_NAME\") | .clusterCrn")

PROVISION_REQUEST=$(jq -n \
  --arg environmentCrn "$ENVCRN" \
  '{
    "instanceGroups": [

    ],
    "environmentCrn": $environmentCrn,
    "tags": [
      {
        "key": "experience",
        "value": "cml-serving"
      }
    ]
  }')

cdp ml create-ml-serving-app \
--app-name $INFNAME \
--environment-crn $ENVCRN \
--cluster-crn $CLUSTERCRN \
--use-public-load-balancer \
--no-skip-validation \
--subnets-for-load-balancers $SUBNET \
--provision-k8s-request $PROVISION_REQUEST
```
