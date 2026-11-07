# Create Cloudera AI service

## Create AI Workbench

### Create a Public CAI workbench
```
WKSPNAME=dyu-ai-wb
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
PROVISION_REQUEST=$(jq -n \
  --arg env "$ENVNAME" \
  --argjson subnets "$(echo "$ENVDETAILS" | jq -c '.environment.network.subnetIds')" \
  '{
    environmentName: $env,
    instanceGroups: [
      {
        name: "liftie-infra",
        instanceType: "m5.4xlarge",
        instanceCount: 2,
        rootVolume: { size: 256 },
        autoscaling: { enabled: true, minInstances: 2, maxInstances: 3 }
      },
      {
        name: "mlinfra",
        instanceType: "m5.4xlarge",
        instanceCount: 2,
        rootVolume: { size: 256 },
        autoscaling: { enabled: true, minInstances: 2, maxInstances: 3 }
      },
      {
        name: "mlcpu0",
        instanceType: "m5.4xlarge",
        instanceCount: 1,
        rootVolume: { size: 256 },
        autoscaling: { enabled: true, minInstances: 0, maxInstances: 10 }
      }
    ],
    network: {
      topology: {
        subnets: $subnets
      }
    }
  }')

cdp ml create-workspace \
--environment-name $ENVNAME \
--workspace-name $WKSPNAME \
--use-public-load-balancer \
--no-private-cluster \
--enable-monitoring \
--enable-governance \
--subnets-for-load-balancers $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
--provision-k8s-request "$PROVISION_REQUEST"
```

### Create a private CAI Workbench
```
WKSPNAME=dyu-ai-wb
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
PROVISION_REQUEST=$(jq -n \
  --arg env "$ENVNAME" \
  --argjson subnets "$(echo "$ENVDETAILS" | jq -c '.environment.network.subnetIds')" \
  '{
    environmentName: $env,
    instanceGroups: [
      {
        name: "liftie-infra",
        instanceType: "m5.4xlarge",
        instanceCount: 2,
        rootVolume: { size: 256 },
        autoscaling: { enabled: true, minInstances: 2, maxInstances: 3 }
      },
      {
        name: "mlinfra",
        instanceType: "m5.4xlarge",
        instanceCount: 2,
        rootVolume: { size: 256 },
        autoscaling: { enabled: true, minInstances: 2, maxInstances: 3 }
      },
      {
        name: "mlcpu0",
        instanceType: "m5.4xlarge",
        instanceCount: 1,
        rootVolume: { size: 256 },
        autoscaling: { enabled: true, minInstances: 0, maxInstances: 10 }
      }
    ],
    network: {
      topology: {
        subnets: $subnets
      }
    }
  }')
cdp ml create-workspace \
--environment-name $ENVNAME \
--workspace-name $WKSPNAME \
--no-use-public-load-balancer \
--private-cluster \
--enable-monitoring \
--enable-governance \
--subnets-for-load-balancers $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
--provision-k8s-request "$PROVISION_REQUEST"
```

## Create AI Registry

### Create Public AI Registry
```
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
CRN=$(echo $ENVDETAILS|jq .environment.crn | tr -d '"')
PROVISION_REQUEST=$(jq -n \
  --argjson subnets "$(echo "$ENVDETAILS" | jq -c '.environment.network.subnetIds')" \
      '{
        "network": {
            "topology": {
                "subnets": $subnets
            }
        }
      }')

cdp ml create-model-registry \
--environment-crn $CRN \
--environment-name $ENVNAME \
--use-public-load-balancer \
--subnets-for-load-balancers $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
--no-private-cluster \
--provision-k8s-request $PROVISION_REQUEST
```

### Create Private AI Registry
```
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
CRN=$(echo $ENVDETAILS|jq .environment.crn | tr -d '"')
PROVISION_REQUEST=$(jq -n \
  --argjson subnets "$(echo "$ENVDETAILS" | jq -c '.environment.network.subnetIds')" \
      '{
        "network": {
            "topology": {
                "subnets": $subnets
            }
        }
      }')

cdp ml create-model-registry \
--environment-crn $CRN \
--environment-name $ENVNAME \
--no-use-public-load-balancer \
--subnets-for-load-balancers $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
--private-cluster \
--provision-k8s-request $PROVISION_REQUEST
```

## Create AI Inference Service
Cloudera AI Inference Service is deployed on environment compute clusters. There are three steps:
1. Create the environment default compute cluster
2. Create another compute cluster
3. Create CAII on a non-default compute cluster

1. Create environment default compute cluster
```
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
WORKER_SUBNETS=$(echo "$ENVDETAILS" | jq -r '.environment.network.subnetIds | join(",")')
cdp environments initialize-aws-compute-cluster \
--environment-name $ENVNAME \
--compute-cluster-configuration privateCluster=true,workerNodeSubnets=$WORKER_SUBNETS
```
2. Create a compute cluster
```
CLUSTERNAME=dyu-compute-cluster
cdp compute create-cluster \
--environment $ENVNAME \
--name $CLUSTERNAME \
--network subnets=$WORKER_SUBNETS
```

3. Deploy the CAII on the compute cluster
NOTE: CAII cannot be deployed on the default cluster. 
```
INFNAME=dyu-inf-svc
ENVCRN=$(echo $ENVDETAILS |jq .environment.crn|tr -d '"')
CLUSTERCRN=$(cdp compute list-clusters | jq -r ".clusters[] | select(.envName == \"$ENVNAME\" and .clusterName == \"$CLUSTERNAME\") | .clusterCrn")

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
```
### For Public Environment
```
cdp ml create-ml-serving-app \
--app-name $INFNAME \
--environment-crn $ENVCRN \
--cluster-crn $CLUSTERCRN \
--use-public-load-balancer \
--no-skip-validation \
--subnets-for-load-balancers $(echo "$ENVDETAILS" | jq -r '.environment.network.subnetIds | join(",")') \
--provision-k8s-request $PROVISION_REQUEST
```
### For Private Environment
```
cdp ml create-ml-serving-app \
--app-name $INFNAME \
--environment-crn $ENVCRN \
--cluster-crn $CLUSTERCRN \
--no-use-public-load-balancer \
--no-skip-validation \
--subnets-for-load-balancers $(echo "$ENVDETAILS" | jq -r '.environment.network.subnetIds | join(",")') \
--provision-k8s-request $PROVISION_REQUEST
```
