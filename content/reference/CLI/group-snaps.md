---
title: Group Snaps using pxctl
keywords: portworx, pxctl, command-line tool, cli, reference
description: Explore the CLI reference guide for taking group snapshots of container data volumes using Portworx.
weight: 7
linkTitle: Group Snaps
---

This document explains how to take group snapshots of your container data with _Portworx_.

First, let's get an overview of the available flags before diving in:

```text
pxctl volume snapshot group -h
```

```
Create group snapshots for given group id or labels

Usage:
  pxctl volume snapshot group [flags]

Aliases:
  group, g

Flags:
  -g, --group string        group id
  -l, --label string        list of comma-separated name=value pairs
  -v, --volume_ids string   list of comma-separated volume IDs
  -h, --help                help for group

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
```

To take a group snapshot of the volumes labelled with `v1=x1`, use this command:

```text
pxctl volume snapshot group --label v1=x1
```

```
Volume 549285969696152595 : Snapshot 1026872711217134654
Volume 952350606466932557 : Snapshot 218459942880193319
```

To take a group snapshot of the volumes created with group `group1`, run:

```text
pxctl volume snapshot group --group “group1”
```

```
Volume 273677465608441312 : Snapshot 609476927441905746

```


[Edit this page on GitHub](https://github.com/portworx/px-docs/blob/gh-pages/control/groupsnap.md)  
