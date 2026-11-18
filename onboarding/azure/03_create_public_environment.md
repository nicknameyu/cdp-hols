# Create Public Environment

## Prepare environment variable
```
CMKMI=$(terraform output -json | jq .managed_ids.value.cmk_ds | tr -d '"')
LOGGERMI=$(terraform output -json | jq .managed_ids.value.logger | tr -d '"')
ASSUMERMI=$(terraform output -json | jq .managed_ids.value.assumer | tr -d '"')
DATAACCESSMI=$(terraform output -json | jq .managed_ids.value.dataaccess | tr -d '"')
RANGERMI=$(terraform output -json | jq .managed_ids.value.ranger | tr -d '"')
RAZMI=$(terraform output -json | jq .managed_ids.value.raz | tr -d '"')

STORAGELOCATION=$(terraform output -json | jq '.storage_locations.value."storage-location-base"'| tr -d '"')
LOGLOCATION=$(terraform output -json | jq '.storage_locations.value."log-location"'| tr -d '"')
BACKUPLOCATION=$(terraform output -json | jq '.storage_locations.value."backup-location"'| tr -d '"')
SSHKEY=$(cat ~/.ssh/id_rsa.pub)

CMKKEYID=$(terraform output -json | jq .cmk_key_id.value|tr -d '"')
CMKRESOURCEGROUP=$(terraform state show 'module.cmk-prerequisite.azurerm_resource_group.kv[0]'|grep " name " | awk -F'"' '{print $2}')

ENVRESOURCEGROUP=$(terraform state show azurerm_resource_group.cdp |grep " name " | awk -F'"' '{print $2}')
VNETINFO=$(terraform show -json | jq '
  .. | objects
  | select(.address? == "module.hub_vnet.azurerm_virtual_network.network")
  | .values
')
VNETNAME=$(echo $VNETINFO|jq -r .name)
NWRESOURCEGROUP=$(echo $VNETINFO|jq -r .resource_group_name)
DEFAULTSG=$(terraform state show 'azurerm_network_security_group.default["default"]'|grep " id "|awk -F'"' '{print $2}' | tr -d '"' )
KNOXSG=$(terraform state show 'azurerm_network_security_group.default["knox"]'|grep " id "|awk -F'"' '{print $2}' | tr -d '"' )
SUBNETS=(
  pub_subnet_1
  pub_subnet_2
  pub_subnet_3
  pub_subnet_4
  pub_subnet_5
  pub_subnet_6
  pub_subnet_7
  pub_subnet_8
)

SUBNET_IDS=$(IFS=,; echo "${SUBNETS[*]}")
PGDBSUBNET=$(terraform state show 'module.spoke_vnet.azurerm_subnet.svc_subnet["pg_flex"]'|grep " name "|grep -v Microsoft | awk -F'"' '{print $2}')

REGION=$(echo $VNETINFO|jq -r .location)

ENVNAME=dyu-az-poc-env
CREDENTIAL=dyu-az-poc
```

## Create environment
```
cdp environments create-azure-environment \
--environment-name $ENVNAME \
--credential-name $CREDENTIAL \
--region $REGION \
--security-access cidr=0.0.0.0/0 \
--workload-analytics \
--enable-tunnel \
--public-key $SSHKEY \
--log-storage storageLocationBase=abfs://${LOGLOCATION},managedIdentity=$LOGGERMI,backupStorageLocationBase=abfs://${BACKUPLOCATION} \
--use-public-ip \
--no-enable-outbound-load-balancer \
--existing-network-params networkId=$VNETNAME,resourceGroupName=$NWRESOURCEGROUP,subnetIds=${SUBNET_IDS} \
--security-access securityGroupIdForKnox=$KNOXSG,defaultSecurityGroupId=$DEFAULTSG \
--free-ipa instanceCountByGroup=2,multiAz=false \
--resource-group-name dyu-cdp-env \
--no-encryption-at-host \
--encryption-key-url $CMKKEYID \
--encryption-key-resource-group-name $CMKRESOURCEGROUP \
--encryption-user-managed-identity $CMKMI
```

## Create IDBroker mapping
```
cdp environments set-id-broker-mappings \
--environment-name $ENVNAME \
--data-access-role $DATAACCESSMI \
--ranger-audit-role $RANGERMI \
--ranger-cloud-access-authorizer-role $RAZMI \
--set-empty-mappings
```

## Create datalake
```
DATALAKENAME=dyu-az-poc-datalake
cdp datalake create-azure-datalake \
--datalake-name $DATALAKENAME \
--environment-name $ENVNAME \
--cloud-provider-configuration managedIdentity=$ASSUMERMI,storageLocation=abfs://${STORAGELOCATION} \
--scale LIGHT_DUTY \
--runtime 7.3.2 \
--no-multi-az \
--enable-ranger-raz 
```

## House keeping commands
- Delete a datalake
```
cdp datalake delete-datalake --datalake-name $DATALAKENAME
```