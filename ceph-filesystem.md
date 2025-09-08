# Deploy Ceph Filesystem
*https://docs.ceph.com/en/latest/cephadm/services/mds/#orchestrator-cli-cephfs*

*https://docs.ceph.com/en/latest/cephfs/client-auth/*

## Create file system volume
```
# ceph fs volume create <fs_name> --placement="<placement spec>"

Example:
root@node-mon01:/home/ubuntu# ceph fs volume create fs-001 --placement="count:3"
root@node-mon01:/home/ubuntu# ceph fs volume create fs-002 --placement="node-mon03,node-mon04,node-mon05"
```

```
root@node-mon01:/home/ubuntu# ceph orch ls
NAME                       PORTS        RUNNING  REFRESHED  AGE   PLACEMENT                         
alertmanager               ?:9093,9094      1/1  65s ago    6h    count:1                           
crash                                       6/6  67s ago    6h    *                                 
grafana                    ?:3000           1/1  65s ago    6h    count:1                           
mds.fs-001                                  3/3  67s ago    74s   count:3                           
mds.fs-002                                  3/3  14s ago    22s   node-mon03;node-mon04;node-mon05  
mds.main                                    6/6  67s ago    12m   *                                 
mgr                                         3/3  67s ago    4h    node-mon01;node-mon02;node-mon03  
mon                                         5/5  67s ago    6h    count:5                           
node-exporter              ?:9100           6/6  67s ago    6h    *                                 
osd.all-available-devices                     6  67s ago    106m  *                                 
prometheus                 ?:9095           1/1  65s ago    6h    count:1

root@node-mon01:/home/ubuntu# ceph fs ls
name: fs-001, metadata pool: cephfs.fs-001.meta, data pools: [cephfs.fs-001.data ]
name: fs-002, metadata pool: cephfs.fs-002.meta, data pools: [cephfs.fs-002.data ]

root@node-mon01:/home/ubuntu# ceph fs volume ls
[
    {
        "name": "fs-001"
    },
    {
        "name": "fs-002"
    }
]
```

![Alt Text](ceph-fs1.png)
![Alt Text](ceph-fs-pool.png)
## Create file system subvolume and set quota limit size storage
> [TIP]
> The --size parameter is actually the number of bytes (<bytes>).
```
# ceph fs subvolume create <fs_name> <subvol_name> --size <bytes>

Example:
root@node-mon01:/home/ubuntu# ceph fs subvolume create fs-001 log --size 5368709120
root@node-mon01:/home/ubuntu# ceph fs subvolume create fs-002 data --size 21474836480
```
![Alt Text](ceph-fs2.png)
![Alt Text](ceph-fs3.png)

## Mount filesystem for client and start storage data
**Install ceph-common**
```
root@ceph-client:/home/ubuntu# apt-get update
root@ceph-client:/home/ubuntu# apt-get install ceph-common
```
