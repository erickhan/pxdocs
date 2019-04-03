---
title: License operations using pxctl
linkTitle: License
keywords: portworx, pxctl, command-line tool, cli, reference, license
description: Learn how to manage licenses using the Portworx CLI.
weight: 8
---

This document explains how to manage your _Portworx_ licenses with
`pxctl license`. The CLI lets you add, activate, and transfer licenses. It also gives details about the installed licenses, and it shows what features are available within a given license.

## Overview

Here's how to get the list of the available subcommands:

```text
pxctl license --help
```

```
Manage licenses

Usage:
  pxctl license [flags]
  pxctl license [command]

Available Commands:
  activate    Activate license from a license server
  add         Add a license from a file
  list        List available licenses
  transfer    Transfer license to remote PX cluster

Flags:
  -h, --help   help for license

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

Use "pxctl license [command] --help" for more information about a command.
```

Now, let's have a closer look at these commands.

## List available licenses

You can use `pxctl license list` to list installed licenses as follows:

```text
pxctl license list
```

```
DESCRIPTION				ENABLEMENT	ADDITIONAL INFO
Number of nodes maximum			1000
Number of volumes maximum		100000
Volume capacity [TB] maximum		  40
Storage aggregation			 yes
Shared volumes				 yes
Volume sets				 yes
BYOK data encryption			 yes
Resize volumes on demand		 yes
Snapshot to object store [CloudSnap]	 yes
Cluster-level migration [PX-Motion]	 yes
Bare-metal hosts			 yes
Virtual machine hosts			 yes
Product SKU				Trial		expires in 6 days, 12:13

LICENSE EXPIRES: 2019-04-07 23:59:59 +0000 UTC
For information on purchase, upgrades and support, see
https://docs.portworx.com/knowledgebase/support.html
```

As you can see, the command gives details on the features allowed under the current licenses and it also lists the SKU.

## Activate a license

The easiest way to activate a license is to get an **activation id** from _Portworx_. Next, run the following:

```text
pxctl license activate <activation-id>
```

However, there are cases where the servers are configured without access to the Internet. Such customers should request an offline-activation license file, and install it like this:

```text
pxctl license add <license file>
```
