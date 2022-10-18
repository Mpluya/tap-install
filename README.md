# tap-install

- kapp deploy -a tap-manager -f ./app -n tap-install 
- kapp deploy -a tap-manager-post-install -f ./post-app -n tap-install 

# tap-install in run cluster
- kapp deploy -a tap-install-gitops -f ./run-app -n tap-install 
- kapp deploy -a run-post-tap-install-gitops -f ./post-app/run-post-app.yml -n tap-install 


## tap-values.yml

Steps to enable external secrets operator using AzureKeyVault as secret store:
1. Install External Secrets Operator (ESO) - https://external-secrets.io/v0.6.0/guides/getting-started/#option-1-install-from-chart-repository
2. Update AKS cluster with OIDC issuer - https://learn.microsoft.com/en-us/azure/aks/cluster-configuration#oidc-issuer
3. Install azwi (azure ad workload identity) cli - https://azure.github.io/azure-workload-identity/docs/installation/azwi.html
4. Follow the quick start guide for Azure AD Workload Identity Federation - https://azure.github.io/azure-workload-identity/docs/quick-start.html. The quick start steps include creating an Azure Key Vault and populating it with secret(s).


    ```
    APPLICATION_NAME="TAP ExtSecret"
    SERVICE_ACCOUNT_NAMESPACE="tap-install"
    SERVICE_ACCOUNT_NAME="eso"
    SERVICE_ACCOUNT_ISSUER="cluster's oidc issuer url from step 2" # or via `az aks show --resource-group [azure resource group name] --name [aks cluster name] --query "oidcIssuerProfile.issuerUrl" -otsv`
    ```

    Create an AAD application and grant permissions to access the secret:
    ```
    azwi serviceaccount create phase app --aad-application-name "${APPLICATION_NAME}"
    ```

    Set access policy for the AAD application or user-assigned managed identity to access the keyvault secret:
    ```
    export APPLICATION_CLIENT_ID="$(az ad sp list --display-name "${APPLICATION_NAME}" --query '[0].appId' -otsv)"
    az keyvault set-policy --name "${KEYVAULT_NAME}" \
      --secret-permissions get \
      --spn "${APPLICATION_CLIENT_ID}"
    ```

    Create a Kubernetes service account and annotate it with the client ID of the AAD application:
    ```
    azwi serviceaccount create phase sa \
    --aad-application-name "${APPLICATION_NAME}" \
    --service-account-namespace "${SERVICE_ACCOUNT_NAMESPACE}" \
    --service-account-name "${SERVICE_ACCOUNT_NAME}"
    ```

    Establish federated identity credential between the AAD application and the service account issuer & subject:
    ```
    azwi serviceaccount create phase federated-identity \
      --aad-application-name "${APPLICATION_NAME}" \
      --service-account-namespace "${SERVICE_ACCOUNT_NAMESPACE}" \
      --service-account-name "${SERVICE_ACCOUNT_NAME}" \
      --service-account-issuer-url "${SERVICE_ACCOUNT_ISSUER}"
    ```
