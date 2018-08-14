# nfs-client-provisioner
Taken from https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client

# Kubernetes NFS-Client Provisioner

[![Docker Repository on Quay](https://quay.io/repository/external_storage/nfs-client-provisioner/status "Docker Repository on Quay")](https://quay.io/repository/external_storage/nfs-client-provisioner)


`nfs-client` is an automatic provisioner that used your *already configured* NFS server, automatically creating Persistent Volumes.

- Persistent volumes are provisioned as ${namespace}-${pvcName}-${pvName}

# How to deploy nfs-client to your cluster.

To note, you must *already* have an NFS Server.

1. Editing:

Modify `deploy.yaml` and change the values to your own NFS server:


```yaml
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 10.10.10.60
            - name: NFS_PATH
              value: /ifs/kubernetes
      volumes:
        - name: nfs-client-root
          nfs:
            server: 10.10.10.60
            path: /ifs/kubernetes
```

Modify `deploy.yaml` to match the same value indicated by `PROVISIONER_NAME`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false" # When set to "false" your PVs will not be archived by the provisioner upon deletion of the PVC.
```

2. Authorization

If your cluster has RBAC enabled, you must authorize the provisioner. If you are in a namespace/project other than "default" either edit `auth.yaml`.

Kubernetes:

```sh
$ kubectl create -f auth.yaml 
serviceaccount "nfs-client-provisioner" created
clusterrole "nfs-client-provisioner-runner" created
clusterrolebinding "run-nfs-client-provisioner" created
```

3. Finally, test your environment!

Now we'll test your NFS provisioner.

Deploy:

```sh
$ kubectl create -f test/test.yaml
```

Now check your NFS Server for the file `SUCCESS`.

```sh
kubectl delete -f test.yaml
```

Now check the folder has been deleted.

4. Deploying your own PersistentVolumeClaim

To deploy your own PVC, make sure that you have the correct `storage-class` as indicated by your `deploy.yaml` file.

For example:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
  annotations:
    volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```