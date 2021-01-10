[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/569/badge)](https://bestpractices.coreinfrastructure.org/projects/569)
## Part 2 - Introduction & Configuration Storage Class & Persistent Volumes (PV/PVC) on OpenShift Advanced Container Platform / Kubernetes
*Written by Kerem ÇELİKER*
- Linkedin: **`linkedin.com/in/keremceliker`**
- Twitter: **`@CloudRss`**
- Blog: **`www.keremceliker.com`**

# How It Works Architecturally PersistentVolume(PV) & PersistentVolumeClaim(PVC)

<img src="https://github.com/keremceliker/Part2-PersistedNFSVolume-Openshift-Kubernetes/blob/main/PV_PVC_Advanced-Concept.png"> 


**Let's scratch and take a look as an example for PV and PVC !**

However..Before start to pressing the gas pedal, I would like to make a few summary and easily-remembered explanations for PV and PVC. I didn't want to explain it in Part 1 :)

Many applications or containers might have storage requirements, so we're going to talk about those. And if you've ever worked with Docker volumes before, we're going to talk about it in the context of Openshift and Kubernetes and how we can use things called persistent volumes, persistent volume claims, and much more... Now, As you know that just about every application out there needs configuration data, so how does that work as you move your app into Kubernetes ? 

We know that pods live and die and so their file system is very short lived or ephemeral, so volumes can be used to store state or data and then use it in a pod so you might be containerizing a database, for example... A pod in the world of OpenShift and Kubernetes can have multiple volumes attached to it, and then containers can create what's called a mountPath to access that volume.

**Persistent Volume:**

A persistent volume is a cluster‑wide storage unit that you can see as the top-above picture and it's provisioned normally by an DevOps & SRE Guy with a lifecycle that's completely independent from the pod. So think of it as something that you as either the DevOps & SRE Guy OR you work with an DevOps & SRE Guy to set up would get in place and then it would talk to Local Storage or cloud storage whatever it may be, huh ? ;)


**Persistent Volume Claim (PVC):**

Now, in order to use one of these, you would use what's called a PersistentVolumeClaim, and this is just a request for a Storage Unit, which would be the persistent volume that would have been set up earlier. So these work hand in hand so that an DevOps & SRE Guy sets up the PV, the persistent volume, and then you might use the PersistentVolumeClaim within your pod or deployment so you can talk to that storage. And here is a breakdown of how this works then. So a persistent volume is cluster‑wide, and this is a resource that is definitely going to rely on some type of network attached storage, now again, that may be cloud, it may be a local network. 

This will rely on a storage provider such as, as mentioned, the network file storage, cloud storage, or maybe some third‑party type of storage option. And then You're going to use this PVC, PersistentVolumeClaim, to kind of say;


**Access Modes - (via K8s Portal):**

A PersistentVolume can be mounted on a host in any way supported by the resource provider. As shown in the table below, providers will have different capabilities and each PV's access modes are set to the specific modes supported by that particular volume. For example, NFS can support multiple read/write clients, but a specific NFS PV might be exported on the server as read-only. Each PV gets its own set of access modes describing that specific PV's capabilities.

```yaml
#The access modes are:**

ReadWriteOnce -- the volume can be mounted as read-write by a single node
ReadOnlyMany -- the volume can be mounted read-only by many nodes
ReadWriteMany -- the volume can be mounted as read-write by many nodes
In the CLI, the access modes are abbreviated to:

RWO - ReadWriteOnce
ROX - ReadOnlyMany
RWX - ReadWriteMany
```

***An IT a Request Animation as usual as below;***

`A Software Developer Guy:`
@Hey Folks ! I have a pod right here... I need to get to this storage over here as soon as possible...
and 

`A DevOps & SRE Guy:`

@Okay,We are going to claim it for you under whichever project you want... 

`Finally, you can use this Storage Volume-PVC (aka LUN) by Storage Pool (PV)`

Now, I am perfectly sure that you got it absolutely all figured out right now ! 
I guess it couldn't be clearer explanation :D

By the way, Now we can get back to work...

I'm definitely not a Database Guy, but I'd like to show you in the simplest way on PostgreSQL on Openshift/Kubernetes/Docker... 

PostgreSQL sequel has persisted our data on to the filesystem but the container file system is not persisted when the container gets destroyed and recreated hence we have some data loss :)

```yaml
keremceliker@bepositive:~$ docker Volume create postgres
```

`docker run -d --rm -v postgres:/var/lib/postgressql/data -e POSTGRES_DB=postgresdb -e POSTGRES_USER=admin -e POSTGRES_PASSWORD:admin123 postgres:10.4`

`keremceliker@bepositive:~$ docker exec -it "Container ID"`

```yaml
psql --username=admin postgresdb
psql (10.4 (Debian 10.4-2.pgdg90+1)
Type "help" for help.

postgresdb-# create a table
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL
);

#show table
\dt

List of relations

Schema | Name | Type | Owner
-----------------------------
public | company | table | admin
(1 row)

# quit 
\q
```

`keremceliker@bepositive:~$ docker rm -f "Container ID"`

Run the same tests as above and notice

`docker run -d --rm -v postges:/var/lib/postgresql/data -e POSTGRES_DB=postgresdb -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin123 postgres:10.4`

You will see the table is being persisted to the file system and is being persisted through of volumes every time they contain the restarts the volumes is reattached to container so we don't "no have any data loss"...

# How does Kubernetes do this ?

Kubernetes has the same concept of volume but it's `called a persistent volume` !

Now the same principle applies to kubernetes so in this example;

`keremceliker@bepositive:~$ cd .\kubernetes/persistentvolumes/`

`keremceliker@bepositive:~$ kubectl create ns postgres`

I'm gonna create a postgre namespace the I'm gonna apply in the postgre namespace an example of a Postgre databases with no persistent volume..

`keremceliker@bepositive:~$ kubectl apply -n postgres -f ./postgres-no-pv.yaml`

This is gonna to deploy a container without any persistent storage whatsoever.
If I apply that we can see have created a postgre stateful state.

`kubectl apply -n postgres -f .\kubernetes\persistentvolumes\postgres-no-pv.yaml`
```yaml
keremceliker@bepositive:~$ kubectl -n postgres get pods

NAME       READY STATUS  RESTARTS AGE
postgre-0   1/1  Running  0        45s
```

Let's go to inside of pod by kubectl exec that I can log into Postgres and create an example table or manage the databases!

`keremceliker@bepositive:~$ kubectl -n postgres exec -it postgres-0 bash`
```yaml
#You will follow the commands in Docker in the same way as the Postgre commands.
```
Now out database is inside the pod and inside the container virtual file system.
So what if I go ahead and destroy that POD ?

`keremceliker@bepositive:~$ kubectl delete po -n postgres postgres-0`

Now you can see kubernetes automatically recreated our pods 
```yaml
NAME       READY STATUS  RESTARTS AGE
postgre-0   1/1  Running  0        45s
```

# If i go back in and login into Postgres, I will print out the tables and you can see the tables have been Lost... WHY ???

It's bloody and perfectly normal :) That's because the container just like in docker has lost its file system when the pod got recreated ;)

Before I run in and show you the details of a persistent volume, lets take a look at what that would look like with Postgres...

I'm gonna create a namespace called Postgres as the follows below:
```yaml
#kubectl create ns postgres
namespace/postgres created
```
I'm quickly gonna create in the Postgre namespace that just gonna create all the resources. 

So I'm gonna create step by step;

```yaml
++ A Persistent Volume (PV)** 
++ A Persistent Volume Claim (PVC)
++ A Create PostGres DB using that Volume
```


If I go in here and apply that and then apply the claim as the follows below:

```yaml
kubectl apply -n postgres -f .\kubernetes\persistentvolumes\persistentvolume.yaml
persistentvolume/example-volume created

kubectl apply -n postgres -f .\kubernetes\persistentvolumes\persistentvolumeclaim.yaml
persistentvolumeclaim/example-volume created
```

**Then I create my Postgres database with a persistent volume as the follows below:**

```yaml
kubectl apply -n postgres -f postgres-with-pv.yaml
configmap/postgres-config created
statefullset.apps/postgres created
service/postgres created
```

**Check the PV and PVC Status**

```yaml
kubectl -n postgres get pv
kubectl -n postgres get pvc

# Check the Postgres PODS
kubectl -n postgres get pods
```

```yaml
Create Postgre DB command transactions like Docker...
```
<img src="https://github.com/keremceliker/Part2-PersistedNFSVolume-Openshift-Kubernetes/blob/main/K8s-Openshift-PVC_on_vSAN"> 

# Let's go ahead and delete that part so we can see if i do get pods kubernetes has gone and recreated our pod...

`kubectl delete po -n postgres postgres-0`

Now under the hood it's mounted that pod to the same persistent volume that has our database so if I go back into the container as the follows below;

`kubectl -n postgres exec -it postgres-0 bash`

I log back into sequel and I list out the tables we can see our tables is there so its gone and mounted to the same persistent that volume and our database has been persistent so we dont have any data loss..

## STORAGE CLASS

=>> persistentvolume.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: qumulo-nfs-volume
  labels:
    type: local
spec:
  #we use local node storage here!
  #kubectl get storageclass
  storageClassName: hostpath
  capacity:
    storage: 1Gi =>> That's going to give us the size of that volume you want to create !
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/Qumulo-NFS-Export/"
```
    
==>> persistentvolumeclaim.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: qumulon-nfs-claim
spec:
  storageClassName: hostpath
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

Storage Classes help Kubernetes interface with the storage as well as describe what the type of storage.

This example here the storage class name is host path and a specific type of kubernetes so if I go to the kubernetes documentation and I go to storage classes if we scroll-down we can see that there's a number of provision is available as a storage class so you can bring an NFS like QUMULO or Azure File/AWS Elastic Block. Cloud and Storage Vendors has own different storages that can bring as a persistent volume. So you need to define either that storage class but many clusters created in the cloud already have a storage classs defines so if you are running Kubernetes on Openshift or Tanzu, Rancher, Nutanix Karbon Kubernetes Platform, you can use a local type storage class to provision by CNI-Plugin...

```yaml
kubectl get storageclasses

Name                 Provisioner             Age
hostpath (default)   docker.io/hostpath      32s
```
For example of an QUMULO NFS Export or Azure Kubernetes Cluster to show you :

```yaml
kubectl get storageclass

NAME                   PROVISIONER                   AGE
qumulofile             kubernetes.io/qumulo-file     45d
qumulofile-nfsexport   kubernetes.io/qumulo-file     45d
default (default)      kubernetes.io/qumulo-file     45d
qumulo-s3              kubernetes.io/qumulo-file     45d

NAME                  PROVISIONER                    AGE
azurefile             kubernetes.io/azure-file       45d
azurefile-premium     kubernetes.io/azure-file       45d
default (default)     kubernetes.io/azure-file       45d
managed-premium       kubernetes.io/azure-file       45d
```
A Notice: That persistent volumes are not bound by namespaces meaning your DevOps & SRE or your full-stack platform engineers that creates this persistent volume can create it for the cluster wide so it's not allocated for a specific namespace that means pods that are running and any namespace can use a persistence volume.

**Hopefully this article helped you understand what a storage class and volume types and to use a persistent volume with a claim.**

# PV & PVC Management Interface on Sample Red Hat Openshift by GUI

**Highly recommend doing it with CLI for all advanced configuration**

<img src="https://github.com/keremceliker/Part2-PersistedNFSVolume-Openshift-Kubernetes/blob/main/Openshift_Example_PVC_Dashboard.png"> 

# A NICE QUESTION ?

Today, I get a good question from a customer, I supported their Kubernetes & OpenShift environment..

**Q:** `Can we bind more than one PersistentVolumeClaim(PVC) into the same PersistentVolume(PV) or 2 PVC ?`

**A:** `No,You can not bind two PVC to in the same PV (Persistent Volume), So you can definitely to use the same in PVC for 2 or 4 different pods.`

* This question has been shared with his permission :)

## Types of Persistent Volumes

PersistentVolume types are implemented as plugins. Kubernetes currently supports the following plugins:

* [`awsElasticBlockStore`](https://kubernetes.io/docs/concepts/storage/volumes/#awselasticblockstore) - AWS Elastic Block Store (EBS)
* [`azureDisk`](https://kubernetes.io/docs/concepts/sotrage/volumes/#azuredisk) - Azure Disk
* [`azureFile`](https://kubernetes.io/docs/concepts/storage/volumes/#azurefile) - Azure File
* [`cephfs`](https://kubernetes.io/docs/concepts/storage/volumes/#cephfs) - CephFS volume
* [`cinder`](https://kubernetes.io/docs/concepts/storage/volumes/#cinder) - Cinder (OpenStack block storage)
  (**deprecated**)
* [`csi`](https://kubernetes.io/docs/concepts/storage/volumes/#csi) - Container Storage Interface (CSI)
* [`fc`](https://kubernetes.io/docs/concepts/storage/volumes/#fc) - Fibre Channel (FC) storage
* [`flexVolume`](https://kubernetes.io/docs/concepts/storage/volumes/#flexVolume) - FlexVolume
* [`flocker`](https://kubernetes.io/docs/concepts/storage/volumes/#flocker) - Flocker storage
* [`gcePersistentDisk`](https://kubernetes.io/docs/concepts/storage/volumes/#gcepersistentdisk) - GCE Persistent Disk
* [`glusterfs`](https://kubernetes.io/docs/concepts/storage/volumes/#glusterfs) - Glusterfs volume
* [`hostPath`](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) - HostPath volume
  (for single node testing only; WILL NOT WORK in a multi-node cluster;
  consider using `local` volume instead)
* [`iscsi`](https://kubernetes.io/docs/concepts/storage/volumes/#iscsi) - iSCSI (SCSI over IP) storage
* [`local`](https://kubernetes.io/docs/concepts/storage/volumes/#local) - local storage devices
  mounted on nodes.
* [`nfs`](https://kubernetes.io/docs/concepts/storage/volumes/#nfs) - Network File System (NFS) storage
* `PhotonPersistentDisk` - Photon controller persistent disk.
  `This volume type no longer works since the removal of the corresponding cloud provider but There is still a hope to release Because this part is currently awaiting triage ! :)`
* [`portworxVolume`](https://kubernetes.io/docs/concepts/storage/volumes/#portworxvolume) - Portworx volume
* [`quobyte`](https://kubernetes.io/docs/concepts/storage/volumes/#quobyte) - Quobyte volume
* [`rbd`](https://kubernetes.io/docs/concepts/storage/volumes/#rbd) - Rados Block Device (RBD) volume
* [`scaleIO`](https://kubernetes.io/docs/concepts/storage/volumes/#scaleio) - ScaleIO volume
  (**deprecated**)
* [`storageos`](https://kubernetes.io/docs/concepts/storage/volumes/#storageos) - StorageOS volume
* [`vsphereVolume`](https://kubernetes.io/docs/concepts/storage/volumes/#vspherevolume) - vSphere VMDK volume


## References:

```yaml

==================================||
https://kubernetes.io/docs/concepts/storage/storage-classes/
==================================||
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
==================================||
https://docs.openshift.com/container-platform/4.6/storage/understanding-persistent-storage.html
==================================||
https://portal.nutanix.com/redirect/page/documents/details?targetId=Kubernetes-Volume-Plug-In-v10:Kubernetes-Volume-Plug-In-v10
==================================||
https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.2/vmware-tanzu-kubernetes-grid-12/GUID-tanzu-k8s-clusters-storage.html
==================================||
https://rancher.com/docs/rancher/v2.x/en/cluster-admin/volumes-and-storage/
==================================||
```
