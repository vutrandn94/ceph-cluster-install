# Deploy Block Device
*https://docs.ceph.com/en/latest/rbd/rados-rbd-cmds/*

## Create a Block Device Pool

```
# ceph osd pool create <pool_name>
# ceph osd pool application enable <pool_name> <application>
# rbd pool init <pool_name>

Example: 
root@node-mon01:~# ceph osd pool create rbd-pool
root@node-mon01:~# ceph osd pool application enable rbd-pool rbd
enabled application 'rbd' on pool 'rbd-pool'
root@node-mon01:~# rbd pool init rbd-pool
```

## Create a Block Device Image
```
# rbd create --size {megabytes} {pool-name}/{image-name}

Example: 
root@node-mon01:~# rbd create --size 10240 rbd-pool/data
```
