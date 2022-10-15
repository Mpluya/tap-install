# tap-install

- kapp deploy -a tap-manager -f ./app -n tap-install 
- kapp deploy -a tap-manager-post-install -f ./post-app -n tap-install 

## tap-values.yml

Steps to enable external secrets operator using AzureKeyVault as secret store:
- Install External Secrets Operator (ESO) - https://external-secrets.io/v0.6.0/guides/getting-started/#option-1-install-from-chart-repository
- Update AKS cluster with OIDC issuer - https://learn.microsoft.com/en-us/azure/aks/cluster-configuration#oidc-issuer
- Install azwi (azure ad workload identity) cli - https://azure.github.io/azure-workload-identity/docs/installation/azwi.html
- Follow the quick start guide for Azure AD Workload Identity Federation - https://azure.github.io/azure-workload-identity/docs/quick-start.html. The quick start steps include creating an Azure Key Vault and populating it with secret(s).


    ```
    APPLICATION_NAME="TAP ExtSecret"
    SERVICE_ACCOUNT_NAMESPACE="tap-install"
    SERVICE_ACCOUNT_NAME="eso"
    ```

    Create an AAD application and grant permissions to access the secret:
    ```
    azwi serviceaccount create phase app --aad-application-name "${APPLICATION_NAME}"
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

- Create an access policy with Azure Key Vault to give the AAD application GET - Secret permission
