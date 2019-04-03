---
title: DC/OS Secrets
logo: /logos/dcos.png
keywords: Portworx, containers, storage, dcos
description: Instructions on using DCOS secrets key management with Portworx
weight: 3
disableprevnext: true
series: key-management
noicon: true
---

Portworx can integrate with DC/OS Secrets to store your encryption keys/secrets and credentials. This guide will help you configure Portworx to connect to DC/OS Secrets. DC/OS Secrets can then be used to store Portworx secrets for Volume Encryption and Cloud Credentials.

{{<info>}}
**Note:**  Secrets is an DC/OS Enterprise only feature
{{</info>}}

{{<info>}}
**Note:**  Supported from PX Enterprise 1.4 onwards
{{</info>}}

## Configuring DC/OS Secrets with Portworx {#configuring-dcos-secrets-with-portworx}

### Configuring permissions for Secrets

To access secrets, Portworx needs credentials of a user. This user should have permissions to access the secrets under a base secrets path. For instance, you can grant permissions to a user to access secrets under `/pwx/secrets` base path, using DC/OS enterprise cli:

```text
dcos security org users grant <username> dcos:secrets:default:/pwx/secrets/* full
```

### Enabling Secrets in Portworx package

During installation or when updating an existing Portworx framework, enable the feature from Secrets section.

![portworx-dcos-secret](/img/dcos-portworx-secrets-setup.png)

The `base path` is the secrets path under which Portworx will read/write secrets. If not specified, Portworx will look for secrets at the top level.

The `dcos username secret` and `dcos password secret` are the paths to secrets, where Portworx will look for credentials of the user to access the secrets. This user should have full access to secrets under the `base path`.


{{<info>}}
**Note:**
If you want Portworx framework to access the username and password secrets path, the path should have prefix same as Portworx service name \(default service name is `portworx`\). Refer [DC/OS docs to know more](https://docs.mesosphere.com/1.12/security/ent/#spaces-for-secrets).
{{</info>}}

## Key generation with DC/OS {#key-generation-with-dcos}

The following sections describe the key generation process with Portworx and DC/OS which can be used for encrypting volumes. For more information about encrypted volumes, [click here](/reference/cli/encrypted-volumes).

### Setting cluster wide secret key

Create a secret in DC/OS using the enterprise cli:

```text
dcos security secrets create --value=<secret-value> pwx/secrets/cluster-wide-secret-key
```

For more details on ways to create Secrets in DC/OS refer [DC/OS documentaion](https://docs.mesosphere.com/1.11/security/ent/secrets/create-secrets)

A cluster wide secret key is a common key that can be used to encrypt all your volumes. You can set the cluster secret key using the following command:

```text
/opt/pwx/bin/pxctl secrets set-cluster-key \
  --secret pwx/secrets/cluster-wide-secret-key
```

This command needs to be run just once for the cluster. If you have added the cluster secret through the config.json, the above command will overwrite it. Even on subsequent Portworx restarts, the cluster secret key in _config.json_ will be ignored for the one set through the CLI.

## \(Optional\) Authenticating with DC/OS Secrets using Portworx CLI

If you do not wish to pass the DC/OS credentials through the framework, you can authenticate Portworx with DC/OS Secrets using Portworx cli. Run the following command:

```text
/opt/pwx/bin/pxctl secrets dcos login \
  --username <dcos-username> \
  --password <dcos-password> \
  --base-path <optional-base-path>
```
{{<info>}}
**Important:**
You need to run this command on all Portworx nodes, so that you could create and mount encrypted volumes on all nodes.
{{</info>}}

If the cli is used to authenticate with DC/OS Secrets, for every restart of Portworx container it needs to be re-authenticated with DC/OS Secrets by running the `dcos login` command.
