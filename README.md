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
docker save influxdb > influxdb-alpine.tar
docker save vmware/powerclicore > powerclicore.tar


[root@icp01-master-1 SW]# docker login mycluster.icp:8500
Username (admin): admin
Password:
Login Succeeded

[root@icp01-master-1 vmware-yamls]# kubectl create -f vmware-namespace.json
namespace/vmware created

[root@icp01-master-1 SW]# docker load -i powerclicore.tar
23dad96f2411: Loading layer [==================================================>]  32.94MB/32.94MB
44af92ee732f: Loading layer [==================================================>]   2.56kB/2.56kB
7700d8c2858c: Loading layer [==================================================>]  244.7MB/244.7MB
98b22a977ecb: Loading layer [==================================================>]  3.473MB/3.473MB
1b516ff64ab6: Loading layer [==================================================>]  1.556MB/1.556MB
2fcce41a368d: Loading layer [==================================================>]   98.3kB/98.3kB
c9ee76c5ec52: Loading layer [==================================================>]  27.65kB/27.65kB
9356a5d439af: Loading layer [==================================================>]  345.9MB/345.9MB
4556bd231b5e: Loading layer [==================================================>]  17.23MB/17.23MB
f1959301da85: Loading layer [==================================================>]  2.596MB/2.596MB
Loaded image: vmware/powerclicore:latest

[root@icp01-master-1 SW]# docker load -i influxdb-alpine.tar
05c2dea6380d: Loading layer [==================================================>]  4.284MB/4.284MB
26e41f27a23a: Loading layer [==================================================>]   2.56kB/2.56kB
8f7356339006: Loading layer [==================================================>]  8.504MB/8.504MB
50187418cdd1: Loading layer [==================================================>]  72.56MB/72.56MB
06b4f5265fa8: Loading layer [==================================================>]  3.072kB/3.072kB
183eaa574760: Loading layer [==================================================>]  2.048kB/2.048kB
187fe7e51b96: Loading layer [==================================================>]  5.632kB/5.632kB
Loaded image: influxdb:1.5.4-alpine


[root@icp01-master-1 SW]# **docker images | grep influx**
	influxdb                                              1.5.4-alpine            e54e3cd3dd62        2 weeks ago         82MB
[root@icp01-master-1 SW]# docker images | grep powercli
	vmware/powerclicore                        latest                      7caca1fba730        4 weeks ago         646MB


[root@icp01-master-1 SW]# **docker tag influxdb:1.5.4-alpine** mycluster.icp:8500/vmware/influxdb:1.5.4-alpine
[root@icp01-master-1 SW]# docker tag vmware/powerclicore:latest mycluster.icp:8500/vmware/powerclicore:ubuntu16.04.0


[root@icp01-master-1 vmware-yamls]# **docker push** mycluster.icp:8500/vmware/influxdb:1.5.4-alpine
The push refers to repository [mycluster.icp:8500/vmware/influxdb]
187fe7e51b96: Pushed
183eaa574760: Pushed
06b4f5265fa8: Pushed
50187418cdd1: Pushed
8f7356339006: Pushed
26e41f27a23a: Pushed
05c2dea6380d: Pushed
1.5.4-alpine: digest: sha256:00e1fbcf66c07a5ff01fe00402b0c592ad151954cda0a04355f87d2ae19c3281 size: 1780

[root@icp01-master-1 vmware-yamls]# docker push mycluster.icp:8500/vmware/powerclicore:ubuntu16.04.0
The push refers to repository [mycluster.icp:8500/vmware/powerclicore]
f1959301da85: Pushed
4556bd231b5e: Pushed
9356a5d439af: Pushed
c9ee76c5ec52: Pushed
2fcce41a368d: Pushed
1b516ff64ab6: Pushed
98b22a977ecb: Pushed
7700d8c2858c: Pushed
44af92ee732f: Pushed
23dad96f2411: Pushed
ubuntu16.04.0: digest: sha256:e5c8558159ca672f5e13dd837c06f185e8c2af5dcf381faa07b77bd45d2fa008 size: 2420


```
