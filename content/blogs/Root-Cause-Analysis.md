+++ 
date = '2026-09-07' 
title = 'First Every Root Cause Analysis' 
tags = ["linux", "kubernetes", "SRE"]
+++

# Root Cause Analysis #1

### Context

This RCA was written as a personal documentation effort to organize the issues and plan for long-term resolution by applying Site Reliability Engineering practices. 

The environment is currently administered and used by a single internal operator to validate configurations before opening the platform to end users. Referneces to "administrator," "operator," and "user" simply describe different roles that the individual (myself) performed during this incident. 

In this analysis, I attempt to follow a blameless approach despite being the sole individual responsible. 

### Incident Definition

A total of 72.6 GB of user data was lost due to corruption of a Longhorn volume and a failed recovery attempt. The incident resulted in a product outage lasting approximately three days, with residual effects continuing for up to seven days due to changes implemented during and after recovery.

### Incident Timeline

#### 17 August

Missing user data was identified in the ownCloud Infinite Scale deployment by the user.

An administrator was alerted and an attempt was made to generate a data snapshot through the Longhorn UI and reconcile the failed volume on a second node.

During this process, the ownCloud user data was stored on only two nodes rather than three. Because the third node did not have sufficient capacity to host the volume, the data became corrupted during a prolonged outage causing cascading failures.

#### 18 August

After multiple attempts to mount the drive into the designated pod, the affected Longhorn volume was determined to be corrupt.

The administrator scaled down the ownCloud deployments to release the associated PVCs. The second copy of the volume was deleted because it was initially believed to be the only corrupt copy, and a manual recovery was attempted using a BusyBox pod.

The recovery attempt failed, resulting in the complete deletion of the PVC and the loss of the remaining usable snapshot.

#### 19 August

Longhorn was replaced with SeaweedFS using the SeaweedFS and SeaweedFS CSI Helm charts to help repidly address storage issues.

All ownCloud Infinite Scale PVCs were reprovisioned using SeaweedFS. This successfully restored application functionality, but the original user data could not be recovered.

User accounts were manually reprovisioned since the accompanying data had already been lost.

The Longhorn backend was temporarily retained in case any additional data migration or recovery work became necessary.

#### 23 August

Additional PVC-related errors were identified during document uploads in ownCloud.

Investigation revealed that a former persistent volume associated with `storageusers-data` was still attached to Longhorn, even though the corresponding PVC reported SeaweedFS as its storage backend.

The remaining Longhorn PV was manually deleted, and an attempt was made to reconcile the ownCloud deployment so that the storage stack would be rebuilt using the new configuration.

#### 24 August

Final reconciliation of the environment was completed.

Flux was found not to be configured for drift detection and therefore did not automatically recreate the PV/PVC resources after their deletion.

The Helm release configuration was manually updated to enable Flux drift detection, reducing the likelihood of similar configuration drift causing future recovery or reconciliation issues.

### Failures and Improvements

#### 1. User-reported failure before administrative detection

The first indication of the storage failure came from an ownCloud user reporting missing data. No automated monitoring or alerting was in place to notify administrators of the Longhorn volume corruption before users were affected.

To improve detection and reduce time to response, Alertmanager should be configured alongside the existing Prometheus and Grafana monitoring stack. Alerts should cover storage health, failed replicas, degraded volumes, unavailable PVCs, and other conditions that could indicate an impending storage failure.

This task is  priority number one and will need to be completed in at most 2 weeks.
#### 2. Longhorn was not well suited to the environment

Longhorn volumes proved overly persistent and operationally difficult to recover within the constraints of the environment, particularly given limited bandwidth, storage capacity, and the relatively small amount of application data being stored.

Longhorn was replaced with SeaweedFS, which is better suited to the low-bandwidth, small-data environment and provides a storage architecture that more closely matches the operational requirements of the cluster.

The new storage solution will need to be re-evalated after some operational time. The administrator will keep logs of all issues relating to SeaweedFS and , on 1 October 2027, will report and failures, gaps, or issues associated with the new solution. 
#### 3. No independent data backup

No independent backup existed for the affected user data. As a result, once the Longhorn volume and available snapshots were lost, there was no secondary recovery source available.

Local application data should be backed up independently of the Kubernetes storage layer. Backups should initially be replicated to a NAS server on the local network and, where feasible, to an external or cloud-based storage provider. The backup process should also include periodic restoration testing to verify that backups are usable during an actual recovery scenario.

This task will occur in two parts. Step one will consist of configuring a local Network Attached Storage as a volume backup target, receiving daily snapshots of all PVCs assocaited to Owncloud. This will be completed no later than 15 October, 2027. Step two will consist of configuring off-site backups for additional reliability. Cost analysis will need to be done in order to detemine the best solution for the given system. A solution will be chosen no later than 31 October, 2027. 

#### 4. No cluster drift detection

Flux was not configured to detect and correct configuration drift. As a result, manually deleted storage resources were not automatically recreated according to the desired cluster state.

Flux drift detection should first be enabled for infrastructure-related workloads and validated to ensure that reconciliation behaves as expected. Once validated, drift detection should also be enabled for application workloads so that manually modified or deleted resources are automatically reconciled back to their declared configuration.

Drift Detection has been temporarily implemented in ownCloud to ensure the system maintains it's healthy state. However, Flux documentation lists limitations of this configuration especially as it relates to pod autoscaling. This will need to be evaluated as a long term solution, but is not a major priority. Completion date will be no later than 30 November, 2027.
