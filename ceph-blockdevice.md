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

## Create a Block Device User

| Client name | Caps | Permissions |   
| :--- | :--- |  :--- | 
| rdp-pool-rw | caps mgr = "profile rbd pool=rbd-pool"<br>caps mon = "profile rbd"<br>caps osd = "profile rbd pool=rbd-pool" | Allow read & write to pool "rbd-pool" and allow client interact RBD features (snapshot, clone,...) |
| :--- | :--- |  :--- | 
| rdp-pool-ro | caps mgr = "profile rbd-read-only pool=rbd-pool"<br>caps mon = "profile rbd"<br>caps osd = "profile rbd-read-only pool=rbd-pool" | Allow read only to pool "rbd-pool" and RBD features (snapshot, clone,...) |

```
root@node-mon01:~# ceph auth get-or-create client.rdp-pool-rw mon 'profile rbd' osd 'profile rbd pool=rbd-pool' mgr 'profile rbd pool=rbd-pool'
[client.rdp-pool-rw]
	key = AQCmib5oM4V1HxAAY9iJAysipnBHFa/dfRYGWA==

root@node-mon01:~# ceph auth get-or-create client.rdp-pool-ro mon 'profile rbd' osd 'profile rbd-read-only pool=rbd-pool' mgr 'profile rbd-read-only pool=rbd-pool'
[client.rdp-pool-ro]
	key = AQDTi75oHibGKRAA2JDrl4r2xGoohKEnrsCskg==

root@node-mon01:~# ceph auth get client.rdp-pool-rw
[client.rdp-pool-rw]
	key = AQCmib5oM4V1HxAAY9iJAysipnBHFa/dfRYGWA==
	caps mgr = "profile rbd pool=rbd-pool"
	caps mon = "profile rbd"
	caps osd = "profile rbd pool=rbd-pool"

root@node-mon01:~# ceph auth get client.rdp-pool-ro
[client.rdp-pool-ro]
	key = AQDTi75oHibGKRAA2JDrl4r2xGoohKEnrsCskg==
	caps mgr = "profile rbd-read-only pool=rbd-pool"
	caps mon = "profile rbd"
	caps osd = "profile rbd-read-only pool=rbd-pool"
```

## Create a Block Device Image
```
# rbd create --size {megabytes} {pool-name}/{image-name}

Example:
root@node-mon01:~# rbd create --size 10240 rbd-pool/data        # image "data" with 10GB quota
```
