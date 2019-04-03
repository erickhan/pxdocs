---
title: Managing cloud credentials using pxctl
keywords: portworx, pxctl, command-line tool, cli, reference
description: Trying to create, list, validate or delete credentials for cloud providers? Follow this step-by-step tutorial from Portworx!
weight: 5
linkTitle: Cloud Credentials
---

## Prerequisites

In this document, we are going to show how to manage your cloud credentials using `pxctl`.

The cloud provider credentials are stored in an external secret store. Before you use the commands from below, you should configure a secret provider of your choice with _Portworx_. For more information, head over to the [Key Management](/key-management) page.

## Overview

You can use the `pxctl credentials` command to create, list, validate, or delete your cloud credentials. Then, _Portworx_ will use these credentials, for example, to back up your volumes to the cloud.

To get a glimpse of the available commands, run:

```text
/opt/pwx/bin/pxctl credentials --help
```

```
Manage credentials for cloud providers

Usage:
  pxctl credentials [flags]
  pxctl credentials [command]

Aliases:
  credentials, cred

Available Commands:
  create      Create a credential for cloud providers
  delete      Delete a credential for cloud
  list        List all credentials for cloud
  validate    Validate a credential for cloud

Flags:
  -h, --help   help for credentials

Global Flags:
      --ca string        path to root certificate for ssl usage
      --cert string      path to client certificate for ssl usage
      --color            output with color coding
      --config string    config file (default is $HOME/.pxctl.yaml)
      --context string   context name that overrides the current auth context
  -j, --json             output in json
      --key string       path to client key for ssl usage
      --raw              raw CLI output for instrumentation
      --ssl              ssl enabled for portworx

Use "pxctl credentials [command] --help" for more information about a command.
```

## List credentials

To list all configured credentials, use this command:

```text
pxctl credentials list
```

```
S3 Credentials
UUID						REGION			ENDPOINT			ACCESS KEY			SSL ENABLED	ENCRYPTION
ffffffff-ffff-ffff-1111-ffffffffffff		us-east-1		s3.amazonaws.com		AAAAAAAAAAAAAAAAAAAA		false		false

Azure Credentials
UUID						ACCOUNT NAME		ENCRYPTION
ffffffff-ffff-ffff-ffff-ffffffffffff		portworxtest		false
```

##  Create and configure credentials

You can create and configure your credentials for various cloud providers as follows:

```text
pxctl credentials create \
  --provider s3 \
  --s3-access-key ***** \
  --s3-secret-key ***** \
  --s3-region us-east-1 \
  --s3-endpoint s3.amazonaws.com
```

```
Credentials created successfully
```

{{<info>}}
**Note:** This command will create a bucket with the _Portworx_ cluster ID to use for the backups.
{{</info>}}

## Delete existing credentials

To delete a particular set of credentials, you can run `pxctl credentials delete` with the `uuid` or the `name` as parameters like this:

```text
pxctl credentials delete <uuid or name>
```

```
Credential deleted successfully
```

{{<info>}}
Don't forget to replace `<uuid or name>` with the actual `uuid` or `name` of the credentials you want to delete.
{{</info>}}


## Validate credentials

If you want to validate a set of credentials for a particular cloud provider, run the following:


```text
pxctl credentials validate <uuid or name>
```

```
Credential validated successfully
```

{{<info>}}
Don't forget to replace `<uuid or name>` with the actual `uuid` or `name` of the credentials you want to delete.
{{</info>}}
