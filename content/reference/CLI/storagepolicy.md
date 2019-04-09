---
title: Storage Policy
description: Manage portworx storage policies
keywords: portworx, storage policy, volume
weight: 1
series: concepts
---

## Overview
The **Storage Policy** feature lets you manage storage policies in a PX cluster with `pxctl storage-policy` or `pxctl stp`. This allows storage system admins to ensure that volumes being created follow a certain default spec. In order to have storage system admin, your token must have access to all groups (`groups: "*"`) and the `system.admin` role.

Storage policies allow for a sort of guardrail against known bad volume specs within your cluster. For example, you can have a storage policy which ensures that volumes created on a PX cluster have a minimum replication level of 2. 

You can also set specific ownership levels for storage policies using the `pxctl stp access` command, which will be covered below. 

## Creating Storage Polices

Create storage policy takes a set of default volume specs to be used during volume creation

```
# pxctl storage-policy create devpol --replication 2,min --secure --sticky --periodic 60,10
 
# pxctl storage-policy list
StoragePolicy   Description
devpol          HA="2,Minimum" Encrypted="true" Sticky="true"...
qapol           HA="2,Minimum" Encrypted="true" Sticky="true"...
```
If the `devpol` storage policy is set to be the **default** storage policy, then all volumes created will have a minimum repl level of 2, encryption enabled, sticky flag on, and periodic snapshot schedule of 60 mins, keeping 10 snapshots. 

## Setting a default storage policy

If a storage policy is set as the default, all volumes created will follow the spec defined by that storage policy. 
You can use the `pxctl storage-policy set-default` command to set a storage policy as the cluster-wide default.

How to set **devpol** as the default storage policy:
```
# pxctl storage-policy set-default devpol
Storage Policy *devpol* is set to default

# pxctl storage-policy list
StoragePolicy   Description
*devpol         Encrypted="true" Sticky="true" SnapInterval="periodic 1h0m0s,keep last 10"...
qapol           HA="2,Minimum" Encrypted="true" Sticky="true"...

# pxctl storage-policy inspect devpol
Storage Policy  :  devpol
    Default                   : Yes
    HA                        : 2,Minimum
    Encrypted                 : true
    Sticky                    : true
    SnapInterval              : periodic 1h0m0s,keep last 10
```
Let's create a volume with a smaller replication level than what's specified in the default storage policy

```
# pxctl v c polvol --repl 1 --size 10
pxctl v i Volume successfully created: 745102698654969688
```
**Note**: The volume should be created with properties repl 2, secure, and snap schedules as periodic 60mins, 10 keeps
```
# ## Inspect Volume ##
# pxctl v i polvol
Volume  :  745102698654969688
    Name                 :  polvol
    Size                 :  10 GiB
    Format               :  ext4
    HA                   :  2
    IO Priority          :  LOW
    Creation time        :  Feb 13 16:41:49 UTC 2019
    Snapshot             :  periodic 1h0m0s,keep last 10
    Shared               :  no
    Status               :  up
    State                :  detached
    Attributes           :  encrypted,sticky
    Reads                :  0
    Reads MS             :  0
    Bytes Read           :  0
    Writes               :  0
    Writes MS            :  0
    Bytes Written        :  0
    IOs in progress      :  0
    Bytes used           :  131 MiB
    Replica sets on nodes:
        Set 0
          Node       : 70.0.82.116 (Pool 0)
          Node       : 70.0.82.114 (Pool 0)
    Replication Status   :  Detached
 ```   

## Removing the default storage policy

To remove a storage policy restriction from PX cluster use `pxctl storage-policy unset-default` .
```
# pxctl storage-policy list
StoragePolicy   Description
*devpol         Encrypted="true" Sticky="true" SnapInterval="periodic 1h0m0s,keep last 10"...
qapol           HA="2,Minimum" Encrypted="true" Sticky="true"...

# ## remove default storage policy restriction ##
# pxctl stp unset-default qapol
Default storage policy restriction is removed

# ## check whether policy is disabled ##
# pxctl storage-policy list
devpol          Encrypted="true" Sticky="true" SnapInterval="periodic 1h0m0s,keep last 10"...
qapol           HA="2,Minimum" Encrypted="true" Sticky="true"...

# ## pxctl storage-policy inspect devpol ##
Storage Policy  :  devpol
    Default                  :  No
    HA                       :  Minimum 2
    Encrypted                :  true
    Sticky                   :  true
    Snapshot             :  periodic 1h0m0s,keep last 10

# pxctl v c nonpol --size 10 --repl 1
Volume successfully created: 880058853866312532
# # Inspect volume
# pxctl v i nonpol
Volume  :  880058853866312532
    Name                 :  nonpol
    Size                 :  10 GiB
    Format               :  ext4
    HA                   :  1
    IO Priority          :  LOW
    Creation time        :  Feb 13 16:51:16 UTC 2019
    Shared               :  no
    Status               :  up
    State                :  detached
    Reads                :  0
    Reads MS             :  0
    Bytes Read           :  0
    Writes               :  0
    Writes MS            :  0
    Bytes Written        :  0
    IOs in progress      :  0
    Bytes used           :  2.6 MiB
    Replica sets on nodes:
        Set 0
          Node       : 70.0.82.116 (Pool 0)
    Replication Status   :  Detached
```

## Updating Storage Policies

You can also update existing storage policy parameters.
eg. Update `qapol` replication from `2,equal to 1,min` 

```
# pxctl stp list    
StoragePolicy Description
prodpol       IOProfile="IO_PROFILE_CMS" SnapInterval="policy=snapSched" HA="2,Equal"...
qapol         SnapInterval="policy=weekpol" HA="2,Minimum" Sticky="true"...
# pxctl stp update qapol --replication 1,min
# pxctl stp list
StoragePolicy Description
prodpol       HA="2,Equal" Encrypted="true" Sticky="true"...
qapol         HA="1,Minimum" Sticky="true" SnapInterval="policy=weekpol"...

```

If a storage policy is updated while set as the default, then volume creation thereafter will follow the updated  policy spec.

```
# pxctl stp list
StoragePolicy Description
prodpol       Sticky="true" IOProfile="IO_PROFILE_CMS" SnapInterval="policy=snapSched"...
*qapol        HA="1,Minimum" Sticky="true" SnapInterval="policy=weekpol"...

# pxctl stp inspect qapol
Storage Policy : qapol
    Default         : Yes
    HA              : 1,Minimum
    Sticky          : true
    SnapInterval    : policy=weekpol

# ## Updating default policy qapol ##
# pxctl stp update qapol --policy snapSched
# pxctl stp inspect qapol
Storage Policy : qapol
        Default         : Yes
        HA              : 1,Minimum
        Sticky          : true
        SnapInterval    : policy=snapSched
```

Let's create volume, it will have **snapSched** as snapshot policy attached.
```
# pxctl v c updatedqapol --size 10
Volume successfully created: 1131539442993682535
# pxctl v i updatedqapol
Volume  :  1131539442993682535
    Name                 :  updatedqapol
    Size                 :  10 GiB
    Format               :  ext4
    HA                   :  1
    IO Priority          :  LOW
    Creation time        :  Feb 19 17:06:53 UTC 2019
    Snapshot             :  policy=snapSched
    Shared               :  no
    Status               :  up
    State                :  detached
    Attributes           :  sticky
    Reads                :  0
    Reads MS             :  0
    Bytes Read           :  0
    Writes               :  0
    Writes MS            :  0
    Bytes Written        :  0
    IOs in progress      :  0
    Bytes used           :  2.6 MiB
    Replica sets on nodes:
        Set 0
          Node       : 70.0.78.114 (Pool 0)
    Replication Status   :  Detached
```

## Deleting Storage Policies

Use `pxctl storage-policy delete <policy-name>` to delete a storage policy. If you want to delete the default policy, the `--force` flag is required.

```
# pxctl stp delete  devpol
Storage Policy devpol is deleted

# ## qapol is default storage policy ##
# pxctl stp delete qapol --force
Storage Policy qapol is deleted
```
**Note**: Deleting default storage policy, will remove volume creation restriction specified by policy

## Create volumes by specifying storage policy
You can also specify any storage policy during volume create: 

```
# pxctl stp create testpol --replication 2,min --sticky --weekly sunday@08:30,8
# pxctl stp list
StoragePolicy   Description
testpol         HA="2,Minimum" Sticky="true" SnapInterval="weekly Sunday@08:30,keep last 8"...
# pxctl stp inspect testpol
Storage Policy  :  testpol
    Default                  :  No
    Sticky                    : true
    SnapInterval              : weekly Sunday@08:30,keep last 8
    HA                        : 2,Minimum
 ```   
Create a volume using storage policy **testpol**

```
# pxctl v c customvol --size 10 --storagepolicy testpol
Volume successfully created: 492212712402729915
[root@ip-70-0-78-110 ~]# pxctl v i customvol
Volume  :  492212712402729915
    Name                 :  customvol
    Size                 :  10 GiB
    Format               :  ext4
    HA                   :  2
    IO Priority          :  LOW
    Creation time        :  Feb 19 17:34:08 UTC 2019
    Snapshot             :  weekly Sunday@08:30,keep last 8
    StoragePolicy        :  testpol
    Shared               :  no
    Status               :  up
    State                :  detached
    Attributes           :  sticky
    Reads                :  0
    Reads MS             :  0
    Bytes Read           :  0
    Writes               :  0
    Writes MS            :  0
    Bytes Written        :  0
    IOs in progress      :  0
    Bytes used           :  2.6 MiB
    Replica sets on nodes:
        Set 0
          Node       : 70.0.78.114 (Pool 0)
          Node       : 70.0.78.110 (Pool 0)
    Replication Status   :  Detached
```
**Note**: 

* Specs such as `replication 2`, snapshot policy `weekly Sunday@08:30,keep last 8`, and `sticky` are all present on the volume inspect
* Specifying a custom policy will override the default storage policy

## Storage Policy options

`pxctl storage policy create --help` shows the available options you can have in a storage policy. A description of how those options can be used is described below.

```
a) If below flags are specified while creating storage policy,  volume creation will have respective spec applied. 
(You need to set default storage policy to make it in affect)

* sticky - sticky volumes cannot be deleted until the flag is disabled
* journal - Journal data for volume
* secure - encrypt volumes using AES-256
* shared - make a globally shared namespace volumes
* aggregation_level string aggregation level (Valid Values: [1 2 3 auto]) (default "1")
* policy string policy names separated by comma
* periodic mins,k periodic snapshot interval in mins,k (keeps 5 by default), 0 disables all schedule snapshots
* daily hh:mm,k daily snapshot at specified hh:mm,k (keeps 7 by default)
* weekly weekday@hh:mm,k weekly snapshot at specified weekday@hh:mm,k (keeps 5 by default)
* monthly day@hh:mm,k monthly snapshot at specified day@hh:mm,k (keeps 12 by default)

b) You can specify min,max or equal replication while creating storage policy. 

eg. :-

1) replication 2,min

If storage policy created with replication 2,min flag. Volume created will be ensured to have replication level at least 2

2) replication 2,max

If storage policy create with replication 2,flag. Volume created will be ensured to have maximum replication specified 2

3) replication 2

If storage policy created with replication 2, Volume created will have exact replication level 2
```


{{<homelist series="px-storage-policy">}}

## Storage policy access control

Storage policies also can have restricted access for specific collaborators and groups. The following commands allow you to update groups and collaborators per storage policy:

* `pxctl stp access add`
* `pxctl stp access remove`
* `pxctl stp access show`
* `pxctl stp access update`

### Storage policy access types
When adding or updating storage policy ACLs, you can provide the following access types:

* __`Read (default)`:__ User or group can use the storage policy
* __`Write`:__ User or group can bypass the storage-policy or update it.
* __`Admin`:__ Can delete the storage-policy (with RBAC access to the StoragePolicy service APIs)


These types can be declared after each group or collaborator name:
```
pxctl stp access add devpol --group group1:w
pxctl stp access add devpol --collaborator collaborator1:a
pxctl stp access add devpol --collaborator collaborator2:r
```

After the above series of commands,

* `group1` will have `Write` access
* `collaborator1` will have `Admin` access
* `collaborator2` will have `Read` access



### Storage policy access update

The update subcommand for storage policies will set the ACLs for that given storage policy. All previous ACLs will be overwritten.

For example, you can update a storage policy to be owned  by a single owner named `user1`:

`pxctl stp access update devpol --owner user1`

Or, you can provide a series of collaborators with access to that storage-policy:

`pxctl stp access update devpol --collaborators user1,user2,user3`

Lastly, you can update a storage-policy to be accessible by a series of groups:

`pxctl stp access update devpol --groups group1,group2`

__Note:__ This command will update all ACLs for a storage-policy. i.e. If you have given access to a series of groups, but do not provide the same groups the next update, those groups will no longer have access.

To add/remove single groups/collaborators to have access, try using `pxctl stp access add/remove`.

### Storage policy access show

To see the ACLs for a given storage-policy, you can use `pxctl stp access show`
```
pxctl stp access show devpol
Storage Policy:  devpol
Ownership:
  Owner:  collaborator1
  Acls:
    Groups:
      group1         Read
      group2         Read
```

### Storage policy access add/remove

To remove or add a single collaborator or group access, you can do so with `pxctl stp access add devpol --collaborator user:w` or `pxctl stp access remove devpol --group group1` 

