# Deploy Block Device
*https://docs.ceph.com/en/latest/rbd/rados-rbd-cmds/*

## Create a Block Device Pool

```
# ceph osd pool create <pool_name>
# ceph osd pool application enable <pool_name> rbd
# rbd pool init <pool_name>

Example: 
root@node-mon01:~# ceph osd pool create rbd-pool
pool 'rbd-pool' created
root@node-mon01:~# ceph osd pool application enable rbd-pool rbd
enabled application 'rbd' on pool 'rbd-pool'
root@node-mon01:~# rbd pool init rbd-pool
root@node-mon01:~# ceph osd pool create rbd-pool-db
pool 'rbd-pool-db' created
root@node-mon01:~# ceph osd pool application enable rbd-pool-db rbd
enabled application 'rbd' on pool 'rbd-pool-db'
root@node-mon01:~# rbd pool init rbd-pool-db
```

## Create a Block Device User

| Client name | Caps | Permissions |   
| :--- | :--- |  :--- | 
| rdp-pool-rw | caps mgr = "profile rbd pool=rbd-pool"<br>caps mon = "profile rbd"<br>caps osd = "profile rbd pool=rbd-pool" | Allow read & write to pool "rbd-pool" and allow client interact RBD features (snapshot, clone,...) |
| rdp-pool-ro | caps mgr = "profile rbd-read-only pool=rbd-pool-db"<br>caps mon = "profile rbd"<br>caps osd = "profile rbd-read-only pool=rbd-pool-db" | Allow read only to pool "rbd-pool-db" and RBD features (snapshot, clone,...) |

```
root@node-mon01:~# ceph auth get-or-create client.rdp-pool-rw mon 'profile rbd' osd 'profile rbd pool=rbd-pool' mgr 'profile rbd pool=rbd-pool'
[client.rdp-pool-rw]
	key = AQCmib5oM4V1HxAAY9iJAysipnBHFa/dfRYGWA==

root@node-mon01:~# ceph auth get-or-create client.rdp-pool-ro mon 'profile rbd' osd 'profile rbd-read-only pool=rbd-pool-db' mgr 'profile rbd-read-only pool=rbd-pool-db'
[client.rdp-pool-ro]
	key = AQCImb5onhurCBAAlxzSsAE0PCbjzfPAQ3KPQw==

root@node-mon01:~# ceph auth get client.rdp-pool-rw
[client.rdp-pool-rw]
	key = AQCmib5oM4V1HxAAY9iJAysipnBHFa/dfRYGWA==
	caps mgr = "profile rbd pool=rbd-pool"
	caps mon = "profile rbd"
	caps osd = "profile rbd pool=rbd-pool"

root@node-mon01:~# ceph auth get client.rdp-pool-ro
[client.rdp-pool-ro]
	key = AQCImb5onhurCBAAlxzSsAE0PCbjzfPAQ3KPQw==
	caps mgr = "profile rbd-read-only pool=rbd-pool-db"
	caps mon = "profile rbd"
	caps osd = "profile rbd-read-only pool=rbd-pool-db"
```

## Create a Block Device Image
```
# rbd create --size {megabytes} {pool-name}/{image-name}

Example:
root@node-mon01:~# rbd create --size 10240 rbd-pool/data        # image "data" with 10GB quota
root@node-mon01:~# rbd create --size 51200 rbd-pool-db/data        # image "data" with 50GB quota
```

## Map RBD image, format and mount to client
**Install ceph-common**
```
root@ceph-client:/home/ubuntu# apt-get update
root@ceph-client:/home/ubuntu# apt-get install ceph-common
root@ceph-client:/home/ubuntu# mkdir -p /ceph-rbd-test/{rbd-pool-rw,rbd-pool-ro}
root@ceph-client:/home/ubuntu# touch /etc/ceph/ceph.keyring && chmod 600 /etc/ceph/ceph.keyring
```

**Config ceph authorize**
> [!NOTE]
> Use command "ceph auth get-key <client user>" to get secret key. Example: "ceph auth get-key client.admin-fs001"
```
--- Get content of file "/etc/ceph/ceph.conf" on 1 mon server node and paste to "/etc/ceph/ceph.conf" on client server or copy file to that ---
root@ceph-client:/home/ubuntu# vi /etc/ceph/ceph.conf
[global]
	fsid = b0c8c6be-8a07-11f0-8f49-7b896d8c3aba
	mon_host = [v2:172.31.24.155:3300/0,v1:172.31.24.155:6789/0] [v2:172.31.29.146:3300/0,v1:172.31.29.146:6789/0] [v2:172.31.17.150:3300/0,v1:172.31.17.150:6789/0] [v2:172.31.24.21:3300/0,v1:172.31.24.21:6789/0] [v2:172.31.17.124:3300/0,v1:172.31.17.124:6789/0]

--- Define client authorize infomation ---
root@ceph-client:/home/ubuntu# vi /etc/ceph/ceph.keyring
[client.rdp-pool-rw]
	key =  AQCmib5oM4V1HxAAY9iJAysipnBHFa/dfRYGWA==
[client.rdp-pool-ro]
	key =  AQCImb5onhurCBAAlxzSsAE0PCbjzfPAQ3KPQw==
```

**Map RBD image, format block device and mount**
> [!NOTE]
> If map with block device pool with read-only permission. You should added option "--read-only"

> [!NOTE]
> Before mount read-only device, you should format device type first (Hint: Mapping to 1 mon server to format with mkfs command) and mount added option "-o ro"

```
root@ceph-client:/home/ubuntu# rbd map data --pool rbd-pool --name client.rdp-pool-rw
/dev/rbd0

root@ceph-client:/home/ubuntu# rbd map data --pool rbd-pool-db --name client.rdp-pool-ro --read-only
/dev/rbd1

root@ceph-client:/home/ubuntu# rbd showmapped
id  pool         namespace  image  snap  device   
0   rbd-pool                data   -     /dev/rbd0
1   rbd-pool-db             data   -     /dev/rbd1

root@ceph-client:~# mkfs.xfs /dev/rbd0
meta-data=/dev/rbd0              isize=512    agcount=16, agsize=163840 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=16     swidth=16 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=16 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.

root@ceph-client:~# mount /dev/rbd0 /ceph-rbd-test/rbd-pool-rw

root@ceph-client:/home/ubuntu# mount -o ro /dev/rbd1 /ceph-rbd-test/rbd-pool-ro

root@ceph-client:~# mount | grep /dev/rbd0
/dev/rbd0 on /ceph-rbd-test/rbd-pool-rw type xfs (rw,relatime,attr2,inode64,logbufs=8,logbsize=64k,sunit=128,swidth=128,noquota)

root@ceph-client:/home/ubuntu# mount | grep /dev/rbd1
/dev/rbd1 on /ceph-rbd-test/rbd-pool-ro type xfs (ro,relatime,attr2,inode64,logbufs=8,logbsize=64k,sunit=128,swidth=128,noquota)

root@ceph-client:~# df -h /dev/rbd0
Filesystem      Size  Used Avail Use% Mounted on
/dev/rbd0        10G  105M  9.9G   2% /ceph-rbd-test/rbd-pool-rw

root@ceph-client:/home/ubuntu# df -h /dev/rbd1
Filesystem      Size  Used Avail Use% Mounted on
/dev/rbd1        50G  928K   50G   1% /ceph-rbd-test/rbd-pool-ro

root@ceph-client:~# echo "123" > /ceph-rbd-test/rbd-pool-rw/test.txt

root@ceph-client:~# cat /ceph-rbd-test/rbd-pool-rw/test.txt
123

root@ceph-client:/home/ubuntu# echo "123" > /ceph-rbd-test/rbd-pool-ro/test.txt
bash: /ceph-rbd-test/rbd-pool-ro/test.txt: Read-only file system
```

## Config auto map device and automount device after server booted
> [!NOTE]
> If config map with block device pool with read-only permission on /etc/ceph/rbdmap , you should added "options='ro'"

```
root@ceph-client:/etc/ceph# cat /etc/ceph/rbdmap 
# RbdDevice		Parameters
#poolname/imagename	id=client,keyring=/etc/ceph/ceph.client.keyring
rbd-pool/data id=rdp-pool-rw,keyring=/etc/ceph/ceph.keyring
rbd-pool-db/data id=rdp-pool-ro,keyring=/etc/ceph/ceph.keyring,options='ro'

root@ceph-client:/etc/ceph# vi /etc/fstab 
/dev/rbd/rbd-pool/data /ceph-rbd-test/rbd-pool-rw xfs defaults,_netdev 0 0
/dev/rbd/rbd-pool-db/data /ceph-rbd-test/rbd-pool-ro xfs defaults,_netdev,ro 0 0
```

> [!NOTE]
> Check systemd service "rbdmap" if device map error. "systemctl status rbdmap"
