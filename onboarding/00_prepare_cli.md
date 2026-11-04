# Command line Authentication

## Install CDP cli: [instruction](https://docs.cloudera.com/cdp-public-cloud/cloud/cli/topics/mc-installing-cdp-client.html)  
## login from CLI
There are two ways to login CDP console from CLI. 
**Option 1:** `cdp login`
1. Find tenant ID on CDP portal->  Management Console -> User name on the nevigation bar -> Profile -> Tenant ID
2. Run command `cdp login --account-id <guid>` to login.

**Option 2:** download profile key
1. Go to CDP portal->  Management Console -> User name on the nevigation bar -> Profile
2. Click `Generate Access Key` to create an access key. 
3. download the access key file and store it to path `~/.cdp/credentials`

