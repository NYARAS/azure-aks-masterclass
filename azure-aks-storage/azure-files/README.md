# Storage Class Notes - Azure Files

[Azure Files](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-introduction). offers fully managed file shares in the cloud that are accessible via the industry standard Server Message Block (SMB) protocol, Network File System (NFS) protocol, and Azure Files REST API.

## Binding Modes

### Immediate (default)
- This setting implies that the PersistentVolumecreation, followed with the storage medium(Azure File in this case) provisioning is triggered as soon as the PersistentVolumeClaim is created.

### WaitForFirstConsumer (Recommended)
- By default, the Immediate mode indicates that volume binding and dynamic provisioning occurs once the PersistentVolumeClaim is created. For storage backends that are topology-constrained and not globally accessible from all Nodes in the cluster, PersistentVolumes will be bound or provisioned without knowledge of the Pod's scheduling requirements. This may result in unschedulable Pods.
A cluster administrator can address this issue by specifying the WaitForFirstConsumer mode which will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created. PersistentVolumes will be selected or provisioned conforming to the topology that is specified by the Pod's scheduling constraints. 

## Reclaim Policy

### Delete (default)
- With this setting, as soon as a PersistentVolumeClaim is deleted, it also triggers the removal of the corresponding PersistentVolume along with the Azure File. This means if you intend to retain the data as backup, then it will not be possible.

### Retain (Recommended)
-  File is retained even when PVC is deleted

### Additional Reference
- https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk

### Managed: 

- When managed used, that file is persisted for the Lifecycle of the cluster. If we delete cluster, it will delete the file

### Example

#### Custom storage class

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile-sc
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
parameters:
  skuName: Standard_LRS
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-azurefile-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: my-azurefile-sc
  resources:
    requests:
      storage: 5Gi
```

#### PVC using AKS provisioned storage class

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-azurefile-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: azurefile # default AKS provisioned storage class in my AKS Cluster
  resources:
    requests:
      storage: 5Gi
