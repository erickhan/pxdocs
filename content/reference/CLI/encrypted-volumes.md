---
title: Encrypted volumes using pxctl
keywords: portworx, px-developer, px-enterprise, plugin, install, configure, container, storage, encryption
description: This guide will give you an overview of how to use the Encryption feature for Portworx volumes.
linkTitle: Encrypted Volumes
weight: 14
---

## Encrypted Volumes

{{% content "shared/encryption/intro.md" %}}

To know more about the supported secret providers and how to configure them with _Portworx_, refer to the [Setup Secrets Provider](/key-management) page.

## Creating and using encrypted volumes

### Using a cluster-wide secret key
A cluster-wide secret key is basically a key-value pair where the value part is the secret that _Portworx_ uses as a passphrase to encrypt all your volumes.

{{<info>}}
Make sure the cluster-wide secret key is set when you are setting up _Portworx_ with one of the supported secret endpoints.
{{</info>}}

Let's look at an example where we want to create and mount an encrypted volume that uses a cluster-wide secret key:

The first step is to create a new volume. Let's make it encrypted with the `--secure` flag:

```text
/opt/pwx/bin/pxctl volume create --secure --size 10 encrypted_volume
```

```
Volume successfully created: 822124500500459627
```

Just to make sure our new encrypted volume was created, try running the following command:

```text
pxctl volume list
```

You should see something like:

```
ID	      	     		NAME		SIZE	HA SHARED	ENCRYPTED	IO_PRIORITY	SCALE	STATUS
822124500500459627	 encrypted_volume	10 GiB	1    no yes		LOW		1	up - detached
```

Next, you can attach the volume:

```text
pxctl host attach encrypted_volume
```

```
Volume successfully attached at: /dev/mapper/pxd-enc822124500500459627
```

We're almost done. Let's mount the volume by running the following command:

```text
pxctl host mount encrypted_volume /mnt
```

```
Volume encrypted_volume successfully mounted at /mnt
```

So, if a cluster-wide secret key is set, _Portworx_ will use it as the default key for encryption.
In the next section, you will learn how to specify per volume keys.


### Using per volume secret keys

As mentioned, you can encrypt volumes using unique keys instead of the cluster-wide secret key. However, you are required to specify the key every time you create or attach a new volume.

Let's look at a simple example. First, we'll run  `pxctl volume create` with the `--secret_key` flag like this:


```text
pxctl volume create --secure --secret_key key1 enc_vol
```

```
Volume successfully created: 374663852714325215
```

Next, mount the `enc_vol` volume into the `mnt` directory as follows:


```text
docker run --rm -it -v secret_key=key1,name=enc_vol:/mnt
```

You can get the same result by typing:

```text
docker run --rm -it --mount src=secret_key=key1,name=enc_vol,dst=/mnt
```

{{<info>}}
Before running the above commands, make sure the secret `key1` exists in the secret endpoint.
{{</info>}}

## Encrypted Shared Volumes

With _Portworx_, you can create encrypted shared volumes that can be accessed from multiple nodes.

The `--shared` flag is used to indicate that we would want to share an encrypted volume:

```bash
pxctl volume create --shared --secure --size 10 encrypted_volume
```

```
Encrypted Shared volume successfully created: 77957787758406722
```

Try inspecting our new volume:

```text
pxctl volume inspect encrypted_volume
```

You should see something like:

```
Volume	:  77957787758406722
Name            	 :  encrypted_volume
Size            	 :  10 GiB
Format          	 :  ext4
HA              	 :  1
IO Priority     	 :  LOW
Creation time   	 :  Nov 1 17:22:59 UTC 2018
Shared          	 :  yes
Status          	 :  up
State           	 :  detached
Attributes      	 :  encrypted
Reads           	 :  0
Reads MS        	 :  0
Bytes Read      	 :  0
Writes          	 :  0
Writes MS       	 :  0
Bytes Written   	 :  0
IOs in progress 	 :  0
Bytes used      	 :  131 MiB
Replica sets on nodes:
	Set 0
		Node 		 : 70.0.18.11 (Pool 0)
Replication Status	 :  Detached
```

You can enable or disable sharing during runtime by passing the `--shared on/off` flag.

Note that volumes must be detached to toggle the `shared` flag during run-time.

The _Portworx_ cluster must be authenticated to access the secret store for the encryption keys.
