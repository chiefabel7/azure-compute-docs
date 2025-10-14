## Intro

Instant Access Snapshot is a new snapshot property on Premium SSD v2 (Pv2) and Ultra disks incremental snapshots. It lets you restore a disk immediately after snapshot creation  - no need to wait for the background data copy to finish. Newly restored disks are rapidly hydrated with minimal performance impact throughout the process.

Instant Access Snapshot is ideal for the following scenarios where speed and performance are critical, such as the following:

1. _Refresh Pre-prod Environment:_ Quickly create new disks from production snapshots to refresh data in testing or training environments.
2. _Rapid Rollback:_ Create snapshots before deploying updates or patches to enable fast recovery from accidental deletions or application-level corruption, minimizing business downtime.
3. _Fast Scale-out:_ Eliminate snapshot creation wait time and rapidly create new disks with the best performance during scale-out events. 

## **About Instant Access Snapshot**

Instant access is a new snapshot attribute available when creating snapshots from **Premium SSD v2** and **Ultra Disks**. Once specified, the snapshot remains in instant access state for the duration defined by InstantAccessDurationMins. After specified instant access duration, it becomes to a Standard Storage snapshot and remains usable. You can monitor its state using the [*SnapshotAccessState*](#) property.

Snapshots in instant access state offer the following key benefits:

1. No need to wait for CompletionPercent to reach 100 before creating new disks from it.
2. New disks created from snapshots are hydrated quickly with minimal performance impact. 

When a snapshot is created with instant access enabled, the system copies the data from the disk to the Standard Storage snapshot in the background. This process is tracked by CompletionPercent property on the snapshot resource.  Until the data is fully copied to Standard Storage, instant access snapshots rely on the availability of the source disk and do not protect against zonal outages. To protect against zonal failures, wait until CompletionPercent reaches 100%. For regional protection, copy the snapshot to another region.

## Restrictions on Instant Access Snapshot

1. Instant access duration must be between 60 and 300 minutes. 
2. Instant access snapshots count towards the limit of 3 in-progress snapshots per disk. 
3. You cannot create instant access snapshots from disks in hydration from another snapshot. Check the _CompletionPercent_ on the source disk — if below 100, hydration is in progress.
4. Instant access snapshots cannot be copied across regions or downloaded until background copy to completes. Track progress via CompletionPercent field on the snapshot.
5. Only Premium SSD v2 and Ultra Disks support instant access snapshots, and only Premium SSD v2 and Ultra disks can be created from them. 
6. You can create up to 15 disks concurrently from instant access snapshots of the same source disk. 
7. You cannot create a larger disk from a smaller instant access snapshot.
8. Attaching a Premium SSD v2 disk across fault domains (e.g., to a VM in an Availability Set or VM Scale Set) triggers background copy, which prevents creation of Instant Access snapshots. Conversely, disks with instant access snapshots cannot be attached across fault domains.
9. You cannot update the encryption property of a disk created from an instant access snapshot while disk hydration is in progress. Similarly, Similarly, disks with instant access snapshots cannot have encryption updates.

Instant access snapshots is supported for REST API (2025-01-02) and CLI (2.77.0), PowerShell (TBD). 

1. Supported regions: France Central

## Billing for instant access snapshot

The following two charges will apply to Instant Access snapshots. (No billing in preview) 

1. _Storage Charge_: Only used space of instant access snapshots will be billed which stores the data change on the source disk after instant access snapshot is created.
2. _Restore Charge_: A one-time operational fee applies when restoring a new disk from an IA snapshot based on the disk size.

## Next Step

[Create Instant Access Snapshot](#)

[Check Snapshot Instant Access State](#)