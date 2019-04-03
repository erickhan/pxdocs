---
title: Run PX-Developer with Docker
keywords: portworx, px-developer, px-enterprise, install, configure, container, storage, add nodes
description: Learn how to run Portworx Developer edition for use with the Docker command line
hidden: true
---

To install and configure _PX Developer Edition_ via the _Docker_ CLI, use the command-line steps described in this section.

{{<info>}}
**Important:** PX stores configuration metadata in a KVDB (key/value store), such as _Etcd_ or _Consul_. We recommend setting up a dedicated kvdb for PX to use. If you want to set one up, see the [etcd example](/reference/knowledge-base/etcd) for PX.
{{</info>}}

### Install and configure Docker

Follow the [Docker install](https://docs.docker.com/engine/installation/) guide to install and start the _Docker_ Service.

### Specify storage

_Portworx_ pools the storage devices on your server and creates a global capacity for containers. The following example uses the two non-root storage devices (/dev/xvdb, /dev/xvdc).

{{<info>}}
**Important:**
Back up any data on storage devices that will be pooled. Storage devices will be reformatted!
{{</info>}}

First, use this command to view the storage devices on your server:

```text
lsblk
```

The output should look similar to:

```
    NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    xvda                      202:0    0     8G  0 disk
    └─xvda1                   202:1    0     8G  0 part /
    xvdb                      202:16   0    64G  0 disk
    xvdc                      202:32   0    64G  0 disk
```

{{<info>}}
Note that devices formatted with a partition are shown under the **TYPE** column as **part**.
{{</info>}}

Next, identify the storage devices you will be allocating to PX. PX can run in a heterogeneous environment, so you can mix and match drives of different types.  Different servers in the cluster can also have different drive configurations.

### Run PX

You can now run PX via the _Docker_ CLI as follows:

```text
if `uname -r | grep -i coreos > /dev/null`; \
then HDRS="/lib/modules"; \
else HDRS="/usr/src"; fi
sudo docker run --restart=always --name px -d --net=host       \
                 --privileged=true                             \
                 -v /run/docker/plugins:/run/docker/plugins    \
                 -v /var/lib/osd:/var/lib/osd:shared           \
                 -v /dev:/dev                                  \
                 -v /etc/pwx:/etc/pwx                          \
                 -v /opt/pwx/bin:/export_bin                   \
                 -v /var/run/docker.sock:/var/run/docker.sock  \
                 -v /var/cores:/var/cores                      \
                 -v ${HDRS}:${HDRS}                            \
                portworx/px-dev -k etcd://myetc.company.com:2379 -c MY_CLUSTER_ID -s /dev/sdb -s /dev/sdc
```

### PX daemon arguments

The following arguments are provided to the PX daemon:

{{% content "install-with-other/docker/shared/cmdargs.md" %}}

### Docker runtime command options

The relevant _Docker_ runtime command options are explained below:

```
--privileged
    > Sets PX to be a privileged container. Required to export block device and for other functions.

--net=host
    > Sets communication to be on the host IP address over ports 9001 -9003. Future versions will support separate IP addressing for PX.

--shm-size=384M
    > PX advertises support for asynchronous I/O. It uses shared memory to sync across process restarts

-v /run/docker/plugins
    > Specifies that the volume driver interface is enabled.

-v /dev
    > Specifies which host drives PX can see. Note that PX only uses drives specified in config.json. This volume flag is an alternate to --device=\[\].

-v /etc/pwx/config.json:/etc/pwx/config.json
    > the configuration file location.

-v /var/run/docker.sock
    > Used by Docker to export volume container mappings.

-v /var/lib/osd:/var/lib/osd:shared
    > Location of the exported container mounts. This must be a shared mount.

-v /opt/pwx/bin:/export_bin
    > Exports the PX command line (**pxctl**) tool from the container to the host.
```

### Optional - running with config.json

You can also provide the runtime parameters to PX via a configuration file called `config.json`.  When this is present, you do not need to pass the runtime parameters via the command line.  This may be useful if you are using tools like _Chef_ or _Puppet_ to provision your host machines.

1.  Download the sample `config.json` file:
https://raw.githubusercontent.com/portworx/px-dev/master/conf/config.json

2.  Create a directory for the configuration file.

    ```text
    # sudo mkdir -p /etc/pwx
    ```

3.  Move the configuration file to that directory. This directory later gets passed in on the Docker command line.

    ```text
    # sudo cp -p config.json /etc/pwx
    ```

4.  Edit the config.json to include the following:
    *   `clusterid`: This string identifies your cluster and must be unique within your etcd key/value space.
    *   `kvdb`: This is the etcd connection string for your etcd key/value store.
    *   `devices`: These are the storage devices that will be pooled from the prior step.


Example config.json:

```json
   {
      "clusterid": "make this unique in your k/v store",
      "dataiface": "bond0",
      "kvdb": [
          "etcd:https://[username]:[password]@[string].dblayer.com:[port]"
        ],
      "mgtiface": "bond0",
      “loggingurl”: “http://dummy:80“,
      "storage": {
        "devices": [
          "/dev/xvdb",
          "/dev/xvdc"
        ]
      }
    }
```

{{<info>}}
**Important:**
If you are using Compose.IO and the `kvdb` string ends with `[port]/v2/keys`, omit the `/v2/keys`. Before running the container, make sure you have saved off any data on the storage devices specified in the configuration.
{{</info>}}

Please also ensure "loggingurl:" is specified in `config.json`. It should either point to a valid lighthouse install endpoint or a dummy endpoint as shown above. This will enable all the stats to be published to monitoring frameworks like _Prometheus_:

You can now start the _Portworx_ container with the following command:

```text
if `uname -r | grep -i coreos > /dev/null`; \
then HDRS="/lib/modules"; \
else HDRS="/usr/src"; fi
sudo docker run --restart=always --name px -d --net=host       \
                 --privileged=true                             \
                 -v /run/docker/plugins:/run/docker/plugins    \
                 -v /var/lib/osd:/var/lib/osd:shared           \
                 -v /dev:/dev                                  \
                 -v /etc/pwx:/etc/pwx                          \
                 -v /opt/pwx/bin:/export_bin                   \
                 -v /var/run/docker.sock:/var/run/docker.sock  \
                 -v /var/cores:/var/cores                      \
                 -v ${HDRS}:${HDRS}                            \
                portworx/px-dev
```

At this point, _Portworx_ should be running on your system. To verify, type:

```text
docker ps
```

#### Authenticated `etcd` and `consul`

To use `etcd` with authentication and a `cafile`, use this in your `config.json`:

```json
"kvdb": [
   "etcd:https://<ip1>:<port>",
   "etcd:https://<ip2>:<port>"
 ],
 "cafile": "/etc/pwx/pwx-ca.crt",
 "certfile": "/etc/pwx/pwx-user-cert.crt",
 "certkey": "/etc/pwx/pwx-user-key.key",
```

To use `consul` with an `acltoken`, use this in your `config.json`:

```json
"kvdb": [
   "consul:http://<ip1>:<port>",
   "consul:http://<ip2>:<port>"
 ],
 "acltoken": "<token>",
```

Alternatively, you could specify and explicit username and password as follows:

```
 "username": "root",
 "password": "xxx",
 "cafile": "/etc/pwx/cafile",
```

### Access the pxctl CLI

Once _Portworx_ is running, you can create and delete storage volumes through the _Docker_ volume commands or the **pxctl** command line tool. With **pxctl**, you can also inspect volumes, the volume relationships with containers, and nodes.

To view all **pxctl** options, run:

```text
/opt/pwx/bin/pxctl help
```

For more information on using **pxctl**, see the [CLI Reference](/reference/cli).

Now, you have successfully setup _Portworx_ on your first server. To increase capacity and enable high availability, repeat the same steps on each of the remaining two servers.

To view the cluster status, run:

```text
/opt/pwx/bin/pxctl status
```

### Adding Nodes

To add nodes in order to increase capacity and enable high availability, simply repeat these steps on other servers. As long as PX is started with the same cluster ID, they will form a cluster.

### Application Examples

Then, to continue with other examples of running stateful applications and databases with _Docker_ and PX, see [this link](/install-with-other/docker/stateful-applications/).
