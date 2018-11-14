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

POWERCLI
https://hub.docker.com/r/vmware/powerclicore/

INFLUXDB
https://hub.docker.com/_/influxdb/


```
    $ docker save influxdb > influxdb-alpine.tar
    $ docker save vmware/powerclicore > powerclicore.tar
```

```
    $ docker login mycluster.icp:8500
      Username (admin): admin
      Password:
      Login Succeeded

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
    1ddsc9b
```

Push docker images to local registry:
```
    $ docker push mycluster.icp:8500/vmware/influxdb:1.5.4-alpine
    $ docker push mycluster.icp:8500/vmware/powerclicore:ubuntu16.04.0
```

## 6. Create and Claim persistent volumes on NFS server, then create Services and Deploy images
