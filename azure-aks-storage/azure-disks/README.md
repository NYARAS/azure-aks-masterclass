# Storage Class Notes - Azure Disk

[Azure managed disks](https://learn.microsoft.com/en-us/azure/virtual-machines/managed-disks-overview) are block-level storage volumes that are managed by Azure and used with Azure Virtual Machines. Managed disks are like a physical disk in an on-premises server but, virtualized. With managed disks, all you have to do is specify the disk size, the disk type, and provision the disk. Once you provision the disk, Azure handles the rest.

## Binding Modes

### Immediate (default)
- This setting implies that the PersistentVolumecreation, followed with the storage medium(Azure Disk in this case) provisioning is triggered as soon as the PersistentVolumeClaim is created.

### WaitForFirstConsumer (Recommended)
- By default, the Immediate mode indicates that volume binding and dynamic provisioning occurs once the PersistentVolumeClaim is created. For storage backends that are topology-constrained and not globally accessible from all Nodes in the cluster, PersistentVolumes will be bound or provisioned without knowledge of the Pod's scheduling requirements. This may result in unschedulable Pods.
A cluster administrator can address this issue by specifying the WaitForFirstConsumer mode which will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created. PersistentVolumes will be selected or provisioned conforming to the topology that is specified by the Pod's scheduling constraints. 

## Reclaim Policy

### Delete (default)
- With this setting, as soon as a PersistentVolumeClaim is deleted, it also triggers the removal of the corresponding PersistentVolume along with the Azure Disk. This means if you intend to retain the data as backup, then it will not be possible.

### Retain (Recommended)
-  Disk is retained even when PVC is deleted

### Additional Reference
- https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk

### Managed: 

- When managed used, that disk is persisted for the Lifecycle of the cluster. If we delete cluster, it will delete the disk

### Example

#### Custom

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium-retain-sc
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain  # Default is Delete, recommended is retain
volumeBindingMode: WaitForFirstConsumer # Default is Immediate, recommended is WaitForFirstConsumer
allowVolumeExpansion: true  
parameters:
  storageaccounttype: Premium_LRS # or use Standard_LRS
  kind: managed
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium-retain-sc
  resources:
    requests:
      storage: 5Gi  
```
