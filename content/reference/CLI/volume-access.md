---
title: Volume access using pxctl
linkTitle: Volume Access
keywords: portworx, container, Kubernetes, storage, role, roles, authorization, authentication, volume, access
description: Learn to update volume access in your px cluster
weight: 3
---

## Overview
This document outlines how to manage your volume access rules. _PX_ allows you to change access permissions per volume. There are two ways to provide access, one is by adding a group and the other is by adding a specific user as a collaborator. When adding a collaborator, you must use the unique id of the user. Please consult with your admin on how to obtain the unique id of the user.

## Access types
When adding a group or collaborator, an access type must also be given, which can be either Read, Write, or Admin:

* __Read__ access type allows access to the volume without modifying it. With a Read access type, one can clone or inspect the volume.
* __Write__ access type allows modification of the volume and its metadata. For example, the user can mount, unmount, and restore the volume from a snapshot in addition to all Read access.
* __Admin__ access type allows full access to the volume, like deleting the volume in addition to all Write access.


## Setting access types
To set access types, add one of the following suffixes to the group or collaborator separated by a colon ':'

* __r__ - For Read access type
* __w__ - For Write access type
* __a__ - For Admin access type

## Volume access commands
_Pxctl_ supports the following commands for updating volume access permissions.

__Add volume access for a single group or collaborator__:

```text
pxctl v access add <volume> --collaborator user1:a
```

__Remove volume access from a group or collaborator__:

```text
pxctl v access remove <volume> --collaborator user1
```

__Show volume access__:

```text
pxctl v access show <volume>
```

__Update full volume access spec__:

```text
pxctl v access update <volume> --groups group1:r,group2:w --collaborators user1:a
```
