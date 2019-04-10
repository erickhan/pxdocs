---
title: "PX-Motion with stork on Kubernetes"
linkTitle: "PX-Motion with stork"
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: How to migrate stateful applications on Kubernetes
series: px-motion
aliases:
  - /cloud-references/migration/migration-stork.html
  - /cloud-references/migration/migration-stork
---

This document will walk you through how to migrate your _PX_ volumes between clusters with Stork on Kubernetes.


## Prerequisites

Before we begin, please make sure the following prerequisites are met:

* **Version**: The source AND destination clusters need _PX-Enterprise_ v2.0 or later
release. As future releases are made, the two clusters can have different _PX-Enterprise_ versions (e.g. v2.1 and v2.3).

* **Stork v2.0+** is required on the source cluster. To install it, pull [this Docker image] (https://hub.docker.com/r/openstorage/stork)

* **Stork helper** : `storkctl` is a command-line tool for interacting with a set of scheduler extensions.
Depending on your operating system, here are the steps you should follow in order to download and install `storkctl`:
  * Linux:

         ```bash
curl http://openstorage-stork.s3-website-us-east-1.amazonaws.com/storkctl/latest/linux/storkctl -o storkctl &&
sudo mv storkctl /usr/local/bin &&
sudo chmod +x /usr/local/bin/storkctl
         ```
  * OS X:

         ```bash
curl http://openstorage-stork.s3-website-us-east-1.amazonaws.com/storkctl/latest/darwin/storkctl -o storkctl &&
sudo mv storkctl /usr/local/bin &&
sudo chmod +x /usr/local/bin/storkctl
         ```
  * Windows:
      * Download [storkctl.exe](http://openstorage-stork.s3-website-us-east-1.amazonaws.com/storkctl/latest/windows/storkctl.exe)
      * Move `storkctl.exe` to a directory in your PATH

* **Secret Store** : Make sure you have configured a [secret store](/key-management) on both clusters. This will be used to store the credentials for the objectstore.

* **Network Connectivity**: Ports 9001 and 9010 on the destination cluster should be
reachable by the source cluster.

## Pairing clusters

On Kubernetes you will define a trust object called **ClusterPair**. This object is required to communicate with the destination cluster. In a nutshell, it creates a pairing with the storage driver (_Portworx_) as well as the scheduler (Kubernetes) so that the volumes and resources can be migrated between clusters.

### Getting the cluster token from the destination cluster

First, let's get the cluster token of the destination cluster. Run the following command from one of the _Portworx_ nodes in the **destination cluster**:


```text
pxctl cluster token show
```

You should see something like:

```
Token is 0795a0bcd46c9a04dc24e15e7886f2f957bfee4146442774cb16ec582a502fdc6aebd5c30e95ff40a6c00e4e8d30e31d4dbd16b6c9b93dfd56774274ee8798cd
```

### Generating the ClusterPair spec

Now that we have the cluster token, the next thing we would want to do is to get the ClusterPair spec from the destination cluster. This is required to migrate _Kubernetes_ resources to the destination cluster.

On the destination cluster, you can generate the template for the spec with this command:

```text
storkctl generate clusterpair -n <migrationnamespace> <remotecluster>
```

{{<info>}}
The name ``<remotecluster>`` is the _Kubernetes_ object that will be created on the source cluster representing the pair relationship. During the actual migration, you will reference this name to identify the destination of your migration.
{{</info>}}

Running the above command should print something like this:

```
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterPair
metadata:
    creationTimestamp: null
    name: remotecluster
    namespace: migrationnamespace
spec:
   config:
      clusters:
         kubernetes:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            certificate-authority-data: <CA_DATA>
            server: https://192.168.56.74:6443
      contexts:
         kubernetes-admin@kubernetes:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            cluster: kubernetes
            user: kubernetes-admin
      current-context: kubernetes-admin@kubernetes
      preferences: {}
      users:
         kubernetes-admin:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            client-certificate-data: <CLIENT_CERT_DATA>
            client-key-data: <CLIENT_KEY_DATA>
    options:
       <insert_storage_options_here>: ""
status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```

### Update ClusterPair with storage options

Next, let's edit the  **ClusterPair** spec. Under `spec.options`, add  the following _Portworx_ clusterpair information:

   1. **ip**: the IP address of one of the _Portworx_ nodes on the destination cluster
   2. **port**: the port on which the _Portworx_ API server is listening for requests.
      Default is 9001 if not specified
   3. **token**: the cluster token generated in the [previous step](#get-cluster-token-from-destination-cluster)

The updated **ClusterPair** should look like this:

```
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterPair
metadata:
  creationTimestamp: null
  name: remotecluster
  namespace: migrationnamespace
spec:
  config:
      clusters:
        kubernetes:
          LocationOfOrigin: /etc/kubernetes/admin.conf
          certificate-authority-data: <CA_DATA>
          server: https://192.168.56.74:6443
      contexts:
        kubernetes-admin@kubernetes:
          LocationOfOrigin: /etc/kubernetes/admin.conf
          cluster: kubernetes
          user: kubernetes-admin
      current-context: kubernetes-admin@kubernetes
      preferences: {}
      users:
        kubernetes-admin:
          LocationOfOrigin: /etc/kubernetes/admin.conf
          client-certificate-data: <CLIENT_CERT_DATA>
          client-key-data: <CLIENT_KEY_DATA>
  options:
      ip: <ip_of_remote_px_node>
      port: <port_of_remote_px_node_default_9001>
      token: <token_generated_from_destination_cluster>>
status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```
Copy and save this to a file called `clusterpair.yaml` on the source cluster.

### Creating the ClusterPair

On the source cluster, create the clusterpair by applying `clusterpair.yaml`:

```text
$ kubectl apply -f clusterpair.yaml
```

```
clusterpair.stork.libopenstorage.org/remotecluster created
```

### Verifying the Pair status

Once you apply the above spec on the source cluster, you should be able to check the status of the pairing:

```text
storkctl get clusterpair
```

On a successful pairing, you should see the "Storage Status" and "Scheduler Status" as "Ready":

```
NAME               STORAGE-STATUS   SCHEDULER-STATUS   CREATED
remotecluster      Ready            Ready              26 Oct 18 03:11 UTC
```

If so, you’re all set and ready to [migrate] (#migrating-volumes-and-resoruces).

### Troubleshooting

If instead, you see an error, you should get more information by running:

```text
kubectl describe clusterpair remotecluster
```

{{<info>}}
You might need to perform additional steps for [GKE](gke) and [EKS](eks)
{{</info>}}

## Migrating Volumes and Resources

Once the pairing is configured, applications can be migrated repeatedly to the destination cluster.

### Starting a migration

#### Using a spec file

In order to make the process schedulable and repeatable, you can write a YAML specification.

In that file, you will specify an object called `Migration`. This object will define the scope of the applications to move and decide whether to automatically start the applications.

Paste this to a file named `migration.yaml`.

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: Migration
metadata:
  name: mysqlmigration
  namespace: migrationnamespace
spec:
  # This should be the name of the cluster pair created above
  clusterPair: remotecluster
  # If set to false this will migrate only the Portworx volumes. No PVCs, apps, etc will be migrated
  includeResources: true
  # If set to false, the deployments and stateful set replicas will be set to 0 on the destination.
  # There will be an annotation with "stork.openstorage.org/migrationReplicas" on the destinationto store the replica count from the source.
  startApplications: true
  # List of namespaces to migrate
  namespaces:
  - migrationnamespace
```

Next, you can invoke this migration manually from the command line:


```text
kubectl apply -f migration.yaml
```

or automate it through `storkctl`:

```text
storkctl create migration mysqlmigration --clusterPair remotecluster --namespaces migrationnamespace --includeResources --startApplications -n migrationnamespace
```

```
Migration mysqlmigration created successfully
```

#### Migration scope

Currently, you can only migrate namespaces in which the object is created. You can also designate one namespace as an admin namespace. This will allow an admin who has access to that namespace to migrate any namespace from the source cluster. Instructions for setting this admin namespace to stork can be found [here](cluster-admin-namespace)

### Monitoring a migration

Once the migration has been started using the above commands, you can check the status using `storkctl`:

```text
storkctl get migration -n migrationnamespace
```

First, you should see something like this:
```
NAME            CLUSTERPAIR     STAGE     STATUS       VOLUMES   RESOURCES   CREATED
mysqlmigration  remotecluster   Volumes   InProgress   0/1       0/0         26 Oct 18 20:04 UTC
```

If the migration is successful, the `Stage` will go from Volumes→ Application→Final.

Here is how the output of a successful migration would look like:

```
NAME            CLUSTERPAIR     STAGE     STATUS       VOLUMES   RESOURCES   CREATED
mysqlmigration  remotecluster   Final     Successful   1/1       3/3         26 Oct 18 20:04 UTC
```

### Troubleshooting

If there is a failure or you want more information about what resources were migrated you can `describe` the migration object using `kubectl`:

```text
$ kubectl describe migration mysqlmigration
```

```
Name:         mysqlmigration
Namespace:    migrationnamespace
Labels:       <none>
Annotations:  <none>
API Version:  stork.libopenstorage.org/v1alpha1
Kind:         Migration
Metadata:
  Creation Timestamp:  2018-10-26T20:04:19Z
  Generation:          1
  Resource Version:    2148620
  Self Link:           /apis/stork.libopenstorage.org/v1alpha1/migrations/ctlmigration3
  UID:                 be63bf72-d95a-11e8-ba98-0214683e8447
Spec:
  Cluster Pair:       remotecluster
  Include Resources:  true
  Namespaces:
      migrationnamespace
  Selectors:           <nil>
  Start Applications:  true
Status:
  Resources:
    Group:      core
    Kind:       PersistentVolume
    Name:       pvc-34bacd62-d7ee-11e8-ba98-0214683e8447
    Namespace:
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       PersistentVolumeClaim
    Name:       mysql-data
    Namespace:  mysql
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      apps
    Kind:       Deployment
    Name:       mysql
    Namespace:  mysql
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
  Stage:        Final
  Status:       Successful
  Volumes:
    Namespace:                mysql
    Persistent Volume Claim:  mysql-data
    Reason:                   Migration successful for volume
    Status:                   Successful
    Volume:                   pvc-34bacd62-d7ee-11e8-ba98-0214683e8447
Events:
  Type    Reason      Age    From   Message
  ----    ------      ----   ----   -------
  Normal  Successful  2m42s  stork  Volume pvc-34bacd62-d7ee-11e8-ba98-0214683e8447 migrated successfully
  Normal  Successful  2m39s  stork  /v1, Kind=PersistentVolume /pvc-34bacd62-d7ee-11e8-ba98-0214683e8447: Resource migrated successfully
  Normal  Successful  2m39s  stork  /v1, Kind=PersistentVolumeClaim mysql/mysql-data: Resource migrated successfully
  Normal  Successful  2m39s  stork  apps/v1, Kind=Deployment mysql/mysql: Resource migrated successfully
```

## Pre and Post Exec rules

Similar to snapshots, a PreExec and PostExec rule can be specified when creating a Migration object. This will result in the PreExec rule being run before the migration is triggered and the PostExec rule to be run after the Migration has been triggered. If the rules do not exist, the Migration will log an event and will stop.

If the **PreExec rule fails** for any reason, it will log an event against the object and retry. **The Migration will not be marked as failed.**

If the **PostExec rule fails** for any reason, it will log an event and **mark the Migration as failed**. It will also try to cancel the migration that was started from the underlying storage driver.

As an example, to add pre and post rules to our migration, we could edit our `migration.yaml` file like this:

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: Migration
metadata:
  name: mysqlmigrationschedule
  namespace: mysql
spec:
  clusterPair: remotecluster
  includeResources: true
  startApplications: false
  preExecRule: mysql-pre-rule
  postExecRule: mysql-post-rule
  namespaces:
  - mysql
```

## Scheduling migrations

{{<info>}}
These features are not yet GA. To test them out, you can use the `openstorage/stork:master` image.
{{</info>}}

### Enabling DR mode

By default, every 7th migration is a full migration. For DR purposes though, we want every migration to be incremental. To enable DR mode, let's update our `migration.yaml` file to store `mode: DisasterRecovery` under `spec.options`:

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterPair
metadata:
  creationTimestamp: null
  name: remotecluster
spec:
  config:
    clusters:
      kubernetes:
        LocationOfOrigin: /etc/kubernetes/admin.conf
        certificate-authority-data: <certificate-authority-data>
        server: https://192.168.56.74:6443
    contexts:
      kubernetes-admin@kubernetes:
        LocationOfOrigin: /etc/kubernetes/admin.conf
        cluster: kubernetes
        user: kubernetes-admin
    current-context: kubernetes-admin@kubernetes
    preferences: {}
    users:
      kubernetes-admin:
        LocationOfOrigin: /etc/kubernetes/admin.conf
        client-certificate-data: <client-certificate-data>
        client-key-data: <client-certificate-data>
  options:
    ip:     <ip_of_remote_px_node>
    port:   <port_of_remote_px_node_default_9001>
    token:  <token_from_step_3>
    mode: DisasterRecovery
status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```

### Schedule policies

You can use schedule policies to specify when a specific action needs to be triggered. Schedule policies do not contain any actions themselves. Also, they are not namespaced.
Storage policies are similar to storage classes where an admin is expected to create schedule policies which are then consumed by other users.

There are 4 sections in a schedule Policy spec:

* **Interval:** the interval in minutes after which the action should be triggered
* **Daily:** the time at which the action should be triggered every day
* **Weekly:** the day of the week and the time in that day when the action should be triggered
* **Monthly:** the date of the month and the time on that date when the action should be triggered

Let's look at an example of how we could spec a policy:

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: SchedulePolicy
metadata:
  name: testpolcy
  namespace: mysql
policy:
  interval:
    intervalMinutes: 1
  daily:
    time: "10:14PM"
  weekly:
    day: "Thursday"
    time: "10:13PM"
  monthly:
    date: 14
    time: "8:05PM"
```

#### Validation

The following validations rules are defined:

* The times in the policy need to follow the time.Kitchen format, example 1:02PM or 1:02pm.
* The date of the month should be greater than 0 and less than 31. If a date doesn't exist in a month, it will roll over to the next month. For example, if the date is specified as Feb 31, it will trigger on either 2nd or 3rd March depending on if it is a leap year.
* The weekday can be specified in either long or short format, ie either "Sunday" or "Sun" are valid days.

{{<info>}}
A policy is not validated when it is created. It is validated when only when it gets associated with a schedule. A policy should not be usable if a field fails validation.
{{</info>}}

### Displaying a policy

To display a policy, run `storkctl get` with the name of the policy as a parameter:

```text
storkctl get schedulepolicy
```

```
NAME           INTERVAL-MINUTES   DAILY     WEEKLY             MONTHLY
testpolicy     1                  10:14PM   Thursday@10:13PM   14@8:05PM
```

### Scheduling a migration

Once a policy has been created, you can use it to schedule a migration. The spec for the MigrationSchedule spec contains the same fields as the Migration spec with the addition of the policy name. The MigrationSchedule object is namespaced like the Migration object.

Note that `startApplications` should be set to false in the spec. Otherwise, the first Migration will start the pods on the remote cluster and will succeed. But all subsequent migrations will fail since the volumes will be in use.

Continuing our previous example with `testpolicy`, here is how to create a `MigrationSchedule` objet that schedules a migration:

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: MigrationSchedule
metadata:
  name: mysqlmigrationschedule
  namespace: mysql
spec:
  template:
    spec:
      clusterPair: remotecluster
      includeResources: true
      startApplications: false
      namespaces:
      - mysql
  schedulePolicyName: testpolicy
```

If the policy name is missing or invalid there will be events logged against the schedule object. Success and failures of the migrations created by the schedule will also result in events being logged against the object. These events can be seen by running a `kubectl describe` on the object

The output of `kubectl describe` will also show the status of the migrations that were triggered for each of the policies along with the start and finish times. The statuses will be maintained for the last successful migration and any Failed or InProgress migrations for each policy type.

Let's now run `kubectl describe` and see how the output would look like:

```text
kubectl describe migrationschedules.stork.libopenstorage.org -n mysql
```

```
Name:         mysqlmigrationschedule

Namespace:    mysql
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"stork.libopenstorage.org/v1alpha1","kind":"MigrationSchedule","metadata":{"annotations":{},"name":"mysqlmigrationschedule",...
API Version:  stork.libopenstorage.org/v1alpha1
Kind:         MigrationSchedule
Metadata:
  Creation Timestamp:  2019-02-14T04:53:58Z
  Generation:          1
  Resource Version:    30206628
  Self Link:           /apis/stork.libopenstorage.org/v1alpha1/namespaces/mysql/migrationschedules/mysqlmigrationschedule
  UID:                 8a245c1d-3014-11e9-8d3e-0214683e8447
Spec:
  Schedule Policy Name:  daily
  Template:
    Spec:
      Cluster Pair:       remotecluster
      Include Resources:  true
      Namespaces:
        mysql
      Post Exec Rule:
      Pre Exec Rule:
      Selectors:           <nil>
      Start Applications:  false
Status:
  Items:
    Daily:
      Creation Timestamp:  2019-02-14T22:16:51Z
      Finish Timestamp:    2019-02-14T22:19:51Z
      Name:                mysqlmigrationschedule-daily-2019-02-14-221651
      Status:              Successful
    Interval:
      Creation Timestamp:  2019-02-16T00:40:52Z
      Finish Timestamp:    2019-02-16T00:41:52Z
      Name:                mysqlmigrationschedule-interval-2019-02-16-004052
      Status:              Successful
      Creation Timestamp:  2019-02-16T00:41:52Z
      Finish Timestamp:    <nil>
      Name:                mysqlmigrationschedule-interval-2019-02-16-004152
      Status:              InProgress
    Monthly:
      Creation Timestamp:  2019-02-14T20:05:41Z
      Finish Timestamp:    2019-02-14T20:07:41Z
      Name:                mysqlmigrationschedule-monthly-2019-02-14-200541
      Status:              Successful
    Weekly:
      Creation Timestamp:  2019-02-14T22:13:51Z
      Finish Timestamp:    2019-02-14T22:16:51Z
      Name:                mysqlmigrationschedule-weekly-2019-02-14-221351
      Status:              Successful
Events:
  Type    Reason      Age                    From   Message
  ----    ------      ----                   ----   -------
  Normal  Successful  4m55s (x53 over 164m)  stork  (combined from similar events): Scheduled migration (mysqlmigrationschedule-interval-2019-02-16-003652) completed successfully
```

Each migration is associated with a Migrations object. To get the most important information, type:

```
kubectl get migration -n mysql
```

```
NAME AGE
mysqlmigrationschedule-daily-2019-02-14-221651 1d
mysqlmigrationschedule-interval-2019-02-16-004052 5m
mysqlmigrationschedule-interval-2019-02-16-004152 4m
mysqlmigrationschedule-monthly-2019-02-14-200541 1d
mysqlmigrationschedule-weekly-2019-02-14-221351 1d
```

Once the MigrationSchedule object is deleted, all the associated Migration objects should also be deleted as well.


## Advanced Operations

* [Migrating to GKE](gke)
* [Migrating to EKS](eks)
* [Configuring a namespace as a cluster namespace](cluster-admin-namespace)
<!--TODO:* [Configuring an external objectstore to be used for migration]-->
