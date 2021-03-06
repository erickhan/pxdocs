---
title: "Production Readiness"
hidden: true
keywords: operations guide, run book, disaster recovery, disaster proof, site failure, node failure, power failure
description: Are you ready for production?
---

### Initial Software Setup for Production

* Deployment - Follow all instructions to deploy Portworx correctly in the scheduler of choice - Refer to the install instructions [page](./)
  * Ensure PX container is deployed as [OCI container](/install-with-other/docker/standalone)
  * All nodes in the cluster should have achieved quorum and `pxctl status` should display the cluster as `operational`
  * etcd - Ensure etcd is properly configured and setup. Setup etcd as a 3-node etcd cluster outside the container orchestrator to ensure maximum stability. Refer to the following [page](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/etcd) on how to install etcd and also configure it for maximum stability.

### Configuring the Server or the Compute Infrastructure

* Ensure that at the minimum 4 cores and 4GB of RAM are allocated for Portworx
* Ensure the base operating system of the server supports linux kernel 3.10+
* Ensure the shared mount propagation is enabled

### Configuring the Networking Infrastructure

* Ensure all the required ports are open as defined in the PX-Enterprise requirements
* Configure separate networks for Data and Management networks to isolate the traffic


### Configuring and Provisioning Underlying Storage

* Disks - If this is a on-prem installation, ensure there is enough storage available per node for PX storage pools.
  If it is a cloud deployment, ensure you have enough cloud disks attached.

  * For AWS ASG, Portworx supports automatic management of EBS volumes.
    It is recommended to use the ASG [feature](/portworx-install-with-kubernetes/cloud/aws/aws-asg)

* HW RAID - If there are a large number of drives in a server and drive failure tolerance is required per server, enable HW RAID (if available) and give the block device from a HW RAID volume for Portworx to manage.

### Volume Management Best Practices

* Volumes - Portworx volumes are thinly provisioned by default. Make sure to monitor for capacity threshold alerts. Refer to the alerts page for more information

### Data Protection for Containers

* Snapshots - Follow DR best practices and ensure volume snapshots are scheduled for instantaneous recovery in the case of app failures. Visit the [DR best practices](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/dr-best-practices) page for more information.

* Cloudsnaps - Follow DR best practices and setup a periodic cloudsnaps so in case of a disaster, Portworx volumes can be restored from an offsite backup

### Alerts and Monitoring for Production

  * Here is how Prometheus can be setup to monitor Portworx [Prometheus](/install-with-other/operate-and-maintain/monitoring/prometheus)
  * Configure Grafana via this [template](/install-with-other/operate-and-maintain/monitoring/grafana)
  * Here is how Alerts Manager can be configured for looking for alerts with [Alerts Manager](/install-with-other/operate-and-maintain/monitoring/alerting)
  * List of Portworx Alerts are documented [here](/install-with-other/operate-and-maintain/monitoring/alerting)

### Hardware Replacements and Upgrades

#### Server Upgrades and Replacements

#### Disk Upgrades and Replacements

#### Networking Upgrades and Replacements

### Software Upgrades

* Portworx Software Upgrades - Work with Portworx Support before planning major upgrades. Ensure all volumes have the
  latest snapshot and cloudsnap before performing upgrades

* Container Orchestrator upgrades - Ensure all volumes are cloud-snapped before performing scheduler upgrades

* OS upgrades - Ensure all volumes have a snapshot before performing underlying OS upgrades.
  Ensure kernel-devel packages are installed after a OS migration
