# Create Data Warehouse on Azure

## Create DW on Public subnets
### Step 1 - prepare parameters

```
CMKMI=$(terraform output -json | jq .managed_ids.value.cmk_ds | tr -d '"')
DWSUBNET=pub_subnet_2
ENVNAME=dyu-az-poc-env
ENVCRN=$(cdp environments describe-environment --environment-name $ENVNAME|jq .environment.crn|tr -d '"')
```

### Step 2 - Create DW
```
cdp dw create-azure-cluster --environment-crn $ENVCRN \
--user-assigned-managed-identity $CMKMI \
--subnet-name $DWSUBNET \
--use-overlay-networking \
--no-use-internal-load-balancer
```


## Create DW on Private subnets
### Step 1 - prepare parameters
- When creating a private DW, several resources must be used: 
  - Postgres DB Delegated subnet has to be used to host the PGDB. 
  - Private DNS zones has to be used for PGDB, AKS
  - A managed identity for CMK as well as AKS admin is optional be recommended. 
    - When using CMK, a User assigned managed identity must be provided.
    - It is recommended to use a User assigned managed identity for the AKS administration. When CMK MI is provided, CDW automatically use the same MI for AKS admin. 
    - Even without CMK, it is also recommended to use the MI for AKS admin, because: if not provided, CDP uses the SPN together with the SPN secret for the AKS admin. When the SPN secret expires, it introduces extra workload to change the password stored in the AKS manually. And users may experience down time for this maintenance. 
```
PGDBDNSZONE=$(terraform output -json |jq -r .private_dns_zones.value.pgdb)
AKSDNSZONE=$(terraform output -json |jq -r .private_dns_zones.value.aks)
CMKMI=$(terraform output -json | jq .managed_ids.value.cmk_ds | tr -d '"')

PGDBSUBNET=$(terraform state show 'module.spoke_vnet.azurerm_subnet.svc_subnet["pg_flex"]'|grep " name "|grep -v Microsoft | awk -F'"' '{print $2}')
DWSUBNET=pvt_subnet_2
ENVNAME=dyu-az-poc-env
ENVCRN=$(cdp environments describe-environment --environment-name $ENVNAME|jq .environment.crn|tr -d '"')
```

### Step 2 - Create DW
```
cdp dw create-azure-cluster --environment-crn $ENVCRN \
--user-assigned-managed-identity $CMKMI \
--subnet-name $DWSUBNET \
--use-overlay-networking \
--use-internal-load-balancer \
--outbound-type "UserDefinedRouting" \
--enable-private-sql \
--private-sql-subnet-name $PGDBSUBNET \
--private-dns-zone-sql $PGDBDNSZONE \
--private-dns-zone-aks $AKSDNSZONE \
--enable-private-aks 
```