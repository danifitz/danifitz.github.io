---
layout: post
title: "Configuring NFS storage for IBM Cloud Private on IBM Cloud Infrastructure"
date: 2018-01-11
---

IBM Cloud Private software is designed to enable companies to create on-premises cloud capabilities similar to public clouds to accelerate app development. The new platform is built on the open source Kubernetes-based container architecture and supports both Docker containers and Cloud Foundry. This facilitates integration and portability of workloads as they evolve to any cloud environment, including the public IBM Cloud.

Because of the ephemeral nature of Containers, the filesystem inside the container is not persisted between container restarts there we need to create a [Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) to store data. This is particularly important when thinking about databases running inside a container.

One of the storage options which is supported by IBM Cloud Private is NFS. We can create a Persistent Volume to make some NFS storage available to our Kubernetes cluster and then when we deploy an application, it can make a [Persistent Volume Claim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) to claim and use that storage.

### Pre-requisites
1. IBM Cloud account, sign up for free [here](https://www.ibm.com/cloud/)
2. IBM Cloud Private cluster, running on IBM Cloud Infrastructure. Get one [here](https://github.com/IBM/deploy-ibm-cloud-private/blob/master/docs/deploy-softlayer-terraform.md)

### Steps we're going to take:
1. Order a File Storage volume from the IBM Cloud Infrastructure portal
2. Mount the NFS storage on our server
3. Create a `PersistentVolume` in IBM Cloud Private
4. Create a `PersistentVolumeClaim` in our application's deployment YAML configuration

### Order File Storage
Go to the IBM Cloud console and expand the menu at the top left, select Infrastructure from the menu.

On the right hand menu select Storage and then File Storage. Choose `Order File Storage`
![selecting storage from the menu]({{ site.url }}/assets/nfs/storage-menu.png)

A wizard will appear and you can select the type, IOPS, Location and Amount of storage you want.
![the storage order form]({{ site.url }}/assets/nfs/storage-order-form.png)

Once you've created the storage and it's been provisioned, select it from the list of Volumes by clicking
on the Volume Name. Scroll down and find the section labelled Authorised Hosts. We need to add the server
that you wish to mount this NFS storage to the list of Authorised hosts.
![adding our server to the list of authorised hosts]({{ site.url }}/assets/nfs/authorised-hosts.png)

Also, note down the value of `Mount Point`, we'll use this value later when mounting the storage.

### Mount the NFS storage on our server

I followed the guide [here](https://console.bluemix.net/docs/infrastructure/FileStorage/mounting-nsf-file-storage.html#mounting-nfs-file-storage) to do this but I've distilled it below.

Next we need to actually mount our NFS storage onto a directory on a server. You can provision a server
whose sole purpose is to have storage mounted on it or like me, you can choose to mount your storage on
one of the IBM Cloud Private nodes. I chose to mount the storage on one of my ICP workers.

First we need to SSH into the machine we are going to mount the storage. You can find the password to your
server by selecting it from the list of devices in the IBM Cloud Infrastructure portal and going to the Password tab.

`ssh root@IP_ADDRESS`

Once you're in, we need to create a `mount` file that describes the NFS mount and then enable it with
`systemctl`.

1. Move directory in the systemd directory `cd /etc/systemd/system`
2. Make a new file with the same filename as the directory you wish to mount. For example if I wanted
to mount my NFS storage at `/home/data` then I would create a file called `home-data.mount`.

```
[Unit]
Description = Mount for NFS storage for ICP PersistentVolume

[Mount]
What=<INSERT_YOUR_STORAGE_MOUNT_POINT>
Where=/home/data
Type=nfs
Options=vers=4,sec=sys,noauto

[Install]
WantedBy = multi-user.target
```

Now enable the mount by running `systemctl enable --now /etc/systemd/system/home-data.mount`

You should be able to verify the success by running `mount | grep home`

### Create a `PersistentVolume` in IBM Cloud Private

Official documentation on creating Persistent Volumes is [here](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0/manage_cluster/create_nfs.html)

Now log into your IBM Cloud Private console and go to Platform > Storage. Click `Create PersistentVolume`.

Fill out the template to create your `PersistentVolume`. Below is how I filled out the template
![creating a persistentvolume]({{ site.url }}/assets/nfs/persistent-volume.png)

```
General:
  Name: my-pv
  Storage class name:
  Capacity: 5 Gi
  Access mode: Read Write once
  Reclaim policy: Retain
  Storage Type: nfs
Labels:
  name: my-pv
  type: nfs
Parameters:
  key: server
  value: HOST_OR_IP_ADDRESS_OF_NFS_SERVER
  key: path
  value: YOUR_NFS_MOUNT_PATH i.e /home/data  
```

### Create a PersistentVolumeClaim in our application's deployment YAML configuration

Now we can create a `PersistentVolumeClaim` in our applications YAML deployment configuration
so that we can dynamically claim the `PersistentVolume` when we deploy our application.

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      name: "my-pv"
```

Here we are using the `matchLabels` selector to match any `PersistentVolume` that has a `Label` which equals
`name=my-pv`. Now in your application's deployment config, when defining a `Volume`, we can reference the
the `PersistentVolumeClaim` we just created.

```
kind: Deployment
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: dockerfile/nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

### Pro-tip
In Kubernetes, a `PersistentVolume` can only have one `PersistentVolumeClaim` at any one time. To avoid
having to have loads of NFS File Storage volumes, you can create sub-directories in your mount path once
you've mounted the storage. For example

```
/home/data
  /db1
  /db2
  /db2
```
  
This will allow us to create multiple `PersistentVolume` per NFS volume.

### Summary
  
So we've created a new NFS File Storage Volume, mounted it to a server, created a `PersistentVolume` in IBM Cloud Private and I've demonstrated how you can claim this storage in your app's deployment config.