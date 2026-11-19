# Create Data Flow

## Create public Data Flow

### Prepare variables
```
ENVNAME=dyu-az-poc-env
SUBNET=pub_subnet_4
ENV_CRN=`cdp environments describe-environment --environment-name $ENVNAME|jq -r .environment.crn`
```

### Run command

```
cdp df enable-service \
  --environment-crn $ENV_CRN \
  --min-k8s-node-count 3 \
  --max-k8s-node-count 20 \
  --use-public-load-balancer \
  --no-private-cluster \
  --cluster-subnets $SUBNET
```

## Create Private Data Flow