# Azure OIDC Login Demo

This repo has an Action that tests logging in to Azure [using OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure).

Full write-up [here](https://colinsalmcorner.com/actions-authenticate-to-azure-without-a-secret/).

For this demo to work, you need 2 SPNs in Azure and 2 environments. The jobs target `dev` and `prod` environments.

# Update: 11/18

You no longer need the composite workflow, since you no longer have to install the `az cli` beta. You can just collapse to this:

```yml
  - uses: azure/login@v1
    name: Log in using OIDC
    with:
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

and you'd be good to go from there!

## Azure Configuration

Steps:
1. Create a `dev` service principal (App Registration) in Azure
2. On the **Certificates and Secrets** tab of the App, click **Federated credentials**
3. Click **+ Add credential** and enter the `org`, `repo` and `environment` (in this case `dev`)
4. On the **Overview** tab, note the `Application (client) ID` for this SPN
5. Give the SPN access to a subscription or Resource Group within the tenant

Repeat these steps for a `prod` SPN, giving it access to a different subscription or set of resource groups.

## GitHub Configuration

Steps:
1. Create a `dev` environment in the repo **Environments** tab under **Settings**
2. Add the `dev` clientID as a secret called `AZURE_CLIENT_ID`

Repeat for the `prod` environment, creating the same secret but use the clientID of the `prod` SPN.

On the repo, configure 2 additional secrets:

- `AZURE_TENANT_ID` - the AAD tenant ID
- `AZURE_SUBSCRIPTION_ID` - the ID of the Azure subscription

> **Note**: If you have different subscriptions for `dev` and `prod`, define the `AZURE_SUBSCRIPTION_ID` at the corresponding environment, rather than sharing a single subscription at the repo level.

## Queue the Workflow

Now you can navigate to the **Actions** tab, click on the `OIDC Demo` workflow and queue it.

## Results

You should see successful deployments to `dev` and `prod`, but the `bad prod` job should fail (I hardcoded the `dev` appID for that job to try to deploy to the `prod` environment with the `dev` SPN).

