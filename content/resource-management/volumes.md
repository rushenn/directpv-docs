---
title: Manage Volumes
date: 2023-05-17
lastmod: :git
draft: false
tableOfContents: true
weight: 30
---

## Prerequisites

* Working DirectPV plugin. 
 
  To install the plugin, refer to the [plugin installation guide]({{< relref "/installation/_index.md#plugin-installation" >}}).
* Working DirectPV CSI driver in Kubernetes. 
 
  To install the driver, refer to the [driver installation guide]({{< relref "/installation/_index.md#driver-installation" >}}).
* Added drives in DirectPV. 
 
  Refer to the [drive management guide]({{< relref "/resource-management/drives.md" >}}).

## Add volume

Refer to the [volume provisioning guide]({{< relref "/resource-management/provisioning.md" >}}).

## List volume

To get information of volumes from DirectPV, run the [`list volumes`]({{< relref "/command-line/list-volumes.md" >}}) command.
The output resembles the following:

```sh
$ kubectl directpv list drives
┌────────┬──────┬──────┬─────────┬─────────┬─────────┬────────┐
│ NODE   │ NAME │ MAKE │ SIZE    │ FREE    │ VOLUMES │ STATUS │
├────────┼──────┼──────┼─────────┼─────────┼─────────┼────────┤
│ master │ vdb  │ -    │ 512 MiB │ 506 MiB │ -       │ Ready  │
│ node1  │ vdb  │ -    │ 512 MiB │ 506 MiB │ -       │ Ready  │
└────────┴──────┴──────┴─────────┴─────────┴─────────┴────────┘
```

## Expand volume

DirectPV supports online volume expansion without requiring a restart of the pods using those volumes. 
This is done automatically after expanding the size for the related `Persistent Volume Claim` (PVC).

1. Get the PVC YAML
   
   kubectl get pvc [PersistentVolumeClaimName] -o yaml > my-file-name.yaml
2. In the PVC, modify `spec.resources.requests.storage` to change the requested size.
3. Apply the updated PVC to the Kubernetes environment, such as with `kubectl apply`.

   After applying the change, the PVC updates and DirectPV automatically increases the size of the volume assigned to the claim.
4. Verify the change with `kubectl get pvc [PersistentVolumeClaimName] -o yaml`

   Review `status.capacity.storage` to see the updated size.

## Delete volume

{{< admonition type="caution" >}}
THIS IS DANGEROUS OPERATION WHICH LEADS TO DATA LOSS
{{< /admonition >}}

A volume can be deleted _only_ if no pod is using the volume _and_ it is in the `Ready` state. 
Run the `kubectl delete pvc` command to trigger DirectPV volume deletion. 
Deleting a volume results in permanent data loss.
Be sure to confirm the volume you are deleting.

```sh
# Delete `sleep-pvc` volume
kubectl delete pvc sleep-pvc
```

## Clean stale volumes
When Pods and/or Persistent Volume Claims are deleted forcefully, associated DirectPV volumes might be left undeleted.
This leads to the volumes becoming stale. 
Remove stale volumes by running the [`clean`]({{< relref "/command-line/clean.md" >}}) command. 

```sh
$ kubectl directpv clean --all
```