# deploy-powercli-influxdb-on-kubernetes-cluster-no-internet

# NOT USEFUL CODE YET - JUST LEARNING....

## Steps
1. Download into your laptop official docker images from internet.
2. Export images into .tar files, and copy them into Master server.
3. Create namespace "vmware"
4. Load, Tag and Push images into local docker registry.
5. Create directories on NFS server to store permanent data.
6. Create and Claim persistent volumes on NFS server, then create Services and Deploy images.
7. Run PowerCli scripts.



## 1. Download into your laptop official docker images from internet
To do that, docker must be installed first (https://docs.docker.com/install/).

Pull docker images:
```
    $ docker pull influxdb:1.5.4-alpine
    $ docker pull vmware/powerclicore
```

Verify docker images:
```
$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
influxdb              1.5.4-alpine        e54e3cd3dd62        2 weeks ago         82MB
vmware/powerclicore   latest              7caca1fba730        4 weeks ago         646MB
```

**IMPORTANT**:
Image's versions change oftently. To check latest available, go to their official web sites:

- powerclicore image
https://hub.docker.com/r/vmware/powerclicore/

- influxdb image (use alpine image edition, which is much lighter than influxdb default one)
https://hub.docker.com/_/influxdb/



## 2. Export docker images into tar files, and copy them into Master server
Export docker images to make them portable:
```
    $ docker save influxdb > influxdb-alpine.tar
    $ docker save vmware/powerclicore > powerclicore.tar
```

Verify .tar files:
```
    $ ls -lh /Users/coco/Docker
      -rw-r--r-- 1 root root 648655872 Nov 14 06:54 powerclicore.tar
      -rw-r--r-- 1 root root  85387776 Nov 14 06:54 influxdb-alpine.tar
```

Then copy these 2 .tar files into "Master" server (ie: under /tmp)
```
  $ ssh root@icp01-master-1
  $ ls -lrt /tmp
    -rw-r--r-- 1 root root 648655872 Nov 14 06:54 powerclicore.tar
    -rw-r--r-- 1 root root  85387776 Nov 14 06:54 influxdb-alpine.tar
```


## 3. Create namespace "vmware"
Login into "Master" server:
```
    $ docker login mycluster.icp:8500
      Username (admin): admin
      Password:
      Login Succeeded
```

Create namespace "vmware"
```
    $ kubectl create -f vmware-namespace.json
      namespace/vmware created

    $ kubectl get namespaces | grep vmware
        vmware         Active    5m

```

## 4. Load, Tag and Push images into "Master" local docker registry

Load docker images locally ("Master" server):
```
    $ docker load -i powerclicore.tar
    $ docker load -i influxdb-alpine.tar
```

Verify loaded images in "Master":
```
    $ docker images | grep influx
      influxdb             1.5.4-alpine    e54e3cd3dd62    2 weeks ago    82MB
    $ docker images | grep powercli
      vmware/powerclicore  latest          7caca1fba730   4 weeks ago    646MB
```

Tag docker image version (optional, but recommended):
```
    $ docker tag influxdb:1.5.4-alpine mycluster.icp:8500/vmware/influxdb:1.5.4-alpine
    $ docker tag vmware/powerclicore:latest mycluster.icp:8500/vmware/powerclicore:ubuntu16.04.0
```
  ie: docker tag imagename:tagname <cluster_CA_domain>:8500/namespacename/imagename:tagname

Verify tags (filtering by "vmware" namespace):
```
    $ docker images | grep vmware
      mycluster.icp:8500/vmware/influxdb     1.5.4-alpine    e54e3cd3dd62    2 weeks ago      82MB
      mycluster.icp:8500/vmware/powerclicore   ubuntu16.04.0   7caca1fba730    4 weeks ago   646MB
```

Push docker images to local registry:
```
    $ docker push mycluster.icp:8500/vmware/influxdb:1.5.4-alpine
    $ docker push mycluster.icp:8500/vmware/powerclicore:ubuntu16.04.0
```

Verification:
If using ICP, at this step, we should see under ICP console these 2 new images:

  Login into ICP Console https://10.71.9.10:8443
  admin / admin

	   Container Images:
		   - vmware/influxdb
		   - vmware/powerclicore

	   Namespaces:
    	 - vmware



## 5. Create directories on NFS server to store permanent data
       ```
         $ ssh root@icp01-nfs-1
         $ mkdir /export/vmware
         $ mkdir /export/vmware/vmware-influxdb-pv
         $ mkdir /export/vmware/vmware-powerclicore-pv
       ```



## 6. Create and Claim persistent volumes on NFS server, then create Services and Deploy images
**IMPORTANT:**
Yes, this can be done through a Helm template, or even merging actions like persistent's volumes creation.
I just run "yaml's" one by one for learning purposes.

Before anything, copy and extract "yamls.tar" file into "Master" server.
```
  $ cp "yamls.tar" root@10.71.9.10:/tmp ("Master" server)

  $ ssh root@icp01-master-1
  $ cd /tmp
  $ tar -xvf yamls.tar
  $ cd /tmp/yamls
```

  <!--- Create "vmware-influxdb-pv" persistent volume (on NFS server) -->
```
  $ kubectl create -f influxdb-pv.yaml
```

  <!--- Claim persistent volume, create service and deploy "vmware-influxdb" image -->
```
  $ kubectl create -f influxdb.yaml
```

  <!--- Create "vmware-powerclicore-pv" persistent volume (on NFS server) -->
```
  $ kubectl create -f powerclicore-pv.yaml
```

  <!--- Claim persistent volume, create service and deploy "vmware-powerclicore" image -->
```
  $ kubectl create -f powerclicore.yaml
```

Verification:
```
  $ kubectl get nodes
```

Access into NFS server and check new files were created under:
```
    - /export/gitlab/gitlab-postgres-pv
    - /export/gitlab/gitlab-redis-pv
    - /export/gitlab/gitlab-pv
```


## 7. Run powercli scripts.
