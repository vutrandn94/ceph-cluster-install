# Integrate CEPH-CSI with K8S
*https://docs.ceph.com/en/reef/rbd/rbd-kubernetes/*

## Create block device pool using for K8S (Perform in 1 ceph mon node)
```
root@node-mon01:/home/ubuntu# ceph osd pool create kubernetes
pool 'kubernetes' created

root@node-mon01:/home/ubuntu# ceph osd pool application enable kubernetes rbd
enabled application 'rbd' on pool 'kubernetes

root@node-mon01:/home/ubuntu# rbd pool init kubernetes
```
## Create a block device user using for K8S (Perform in 1 ceph mon node)
```
root@node-mon01:/home/ubuntu# ceph auth get-or-create client.kubernetes mon 'profile rbd' osd 'profile rbd pool=kubernetes' mgr 'profile rbd pool=kubernetes'
[client.kubernetes]
	key = AQDH5r9o/UKABRAAD0Oz3roseMT9sSD6cNBtRw==
```

## Collect the Ceph cluster unique fsid and the monitor addresses (Perform in 1 ceph mon node)
```
root@node-mon01:/home/ubuntu# ceph mon dump
epoch 5
fsid b0c8c6be-8a07-11f0-8f49-7b896d8c3aba
last_changed 2025-09-05T04:30:30.832262+0000
created 2025-09-05T03:23:34.207636+0000
min_mon_release 17 (quincy)
election_strategy: 1
0: [v2:172.31.24.155:3300/0,v1:172.31.24.155:6789/0] mon.node-mon01
1: [v2:172.31.29.146:3300/0,v1:172.31.29.146:6789/0] mon.node-mon02
2: [v2:172.31.17.150:3300/0,v1:172.31.17.150:6789/0] mon.node-mon03
3: [v2:172.31.24.21:3300/0,v1:172.31.24.21:6789/0] mon.node-mon04
4: [v2:172.31.17.124:3300/0,v1:172.31.17.124:6789/0] mon.node-mon05
dumped monmap epoch 5
```

## Generate ceph-csi manifest and auto deploy (Perform in k8s client)
