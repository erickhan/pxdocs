---
title: Azure Key Vault
logo: /logos/azure-log.png
keywords: portworx, containers, storage, vault
description: Instructions on using Azure key vault(Secrets) with Portworx
weight: 7
disableprevnext: true
series: key-management
noicon: true
---

Portworx can integrate with Azure Key Vault [Secrets](https://docs.microsoft.com/en-us/azure/key-vault/about-keys-secrets-and-certificates#key-vault-secrets) to store your encryption secrets, credentials or passwords. This guide will get a Portworx cluster connected to a Azure Key Vault. The Azure Key Vault could be used to store secrets that will be used for encrypting volumes.

## Setting up Azure Key Vault {#setting-up-azure-kv}

Peruse [this section](https://docs.microsoft.com/en-us/azure/key-vault/quick-create-portal) for help on setting up Azure Key Vault in your setup. 
You will also require to register and authenticate application with Azure Key Vault. 

- Please follow [doc] (https://docs.microsoft.com/en-us/azure/key-vault/key-vault-manage-with-cli2#registering-an-application-with-azure-active-directory) to register application with azure active directory 
- Please follow [doc](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-manage-with-cli2#authorizing-an-application-to-use-a-key-or-secret) to grant Azure Key Vault permission to your registered app.

Portworx will need application that has Azure Key Vault `set/get/list/delete` secrets permissions. 

Following are the authentication details required by Portworx to connect Azure Key Vault -

- `AZURE_VAULT_URL`: Azure Key Vault URL 
- `AZURE_TENANT_ID`: Azure Active Directory ID
- `AZURE_CLIENT_ID`: Azure application ID which is registered with Azure active directory and has access to azure key vault mentioned in `AZURE_VAULT_URL`
- `AZURE_CLIENT_SECRET`: Azure application secret id, you may need to generate one if not created already. Follow [doc](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal#get-application-id-and-authentication-key) to generate new secret key for your application
- `AZURE_ENVIRONMENT`(optional): Azure environment or [az cloud list](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest).  Default cloud name is AzurePublicCloud

### Kubernetes users {#kubernetes-users}

If you are installing Portworx on Kubernetes, when generating the [Portworx Kubernetes spec file](https://install.portworx.com/):

1. Pass in all the above variables as is in the Environment Variables section.
2. Select `Azure Key Vault` from the `Secrets Store Type` list under `Advanced Settings`

To generate Portworx spec for Kubernetes, refer instructions, [click here](/portworx-install-with-kubernetes).


### Other users {#other-users}

#### New installation

During installation,

1. Use argument `-secret_type azure-kv -cluster_secret_key <secret-id>` when starting Portworx to specify the secret type as vault and the cluster-wide secret key.
2. Use `-e` docker option to expose the Azure Key Vault enviornment variables.

#### Existing installation

Based on your installation method provide the `-secret_type azure-kv` input argument and environment variable and restart PX on all the nodes.

{{<info>}}
**Note**: 
<br>
Portworx supports only the [Azure KeyVault Secrets](https://docs.microsoft.com/en-us/azure/key-vault/about-keys-secrets-and-certificates#key-vault-secrets)
<br>
Portworx does not support [Azure KeyVault Keys soft/hard](https://docs.microsoft.com/en-us/azure/key-vault/about-keys-secrets-and-certificates#key-vault-keys)
{{</info>}}

### Setting cluster wide secret key

A cluster wide secret key is a common key that can be used to encrypt all your volumes. You can set the cluster secret key using the following command.

```text
/opt/pwx/bin/pxctl secrets set-cluster-key --secret <cluster-wide-secret-key>
```

This command needs to be run just once for the cluster. If you have added the cluster secret key through the config.json, the above command will overwrite it. Even on subsequent Portworx restarts, the cluster secret key in config.json will be ignored for the one set through the CLI.

{{<info>}}
**Important:**
Make sure that the secret key has been created in Azure Key Vault [Secrets](https://docs.microsoft.com/en-us/azure/key-vault/about-keys-secrets-and-certificates#key-vault-secrets).
{{</info>}}
