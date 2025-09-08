# Deploy Block Device
*https://docs.ceph.com/en/latest/rbd/rados-rbd-cmds/*

## Create a Block Device Pool
```
root@node-mon01:~# ceph osd pool create rbd-pool
root@node-mon01:~# ceph osd pool application enable rbd-pool rbd
enabled application 'rbd' on pool 'rbd-pool'
root@node-mon01:~# rbd pool init rbd-pool
```
