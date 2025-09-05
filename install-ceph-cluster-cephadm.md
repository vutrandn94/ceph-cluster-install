# Deploy Ceph cluster with cephadm

| Requirements |
| :--- |
| Python 3 |
| Systemd |
| Podman or Docker for running containers |
| Time synchronization (such as Chrony or the legacy ntpd) |
| LVM2 for provisioning storage devices |


## Lab info (6 servers - 5 mon + osd, 1 osd)
| Hostname | IP Address | OS | Role | Label | Disk Device Storage Data (50 GB) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| node-mon01 | 172.31.24.155  | Ubuntu 22.04.5 LTS | mon, osd, mgr | _admin | /dev/xvdb |
| node-mon02 | 172.31.29.146 | Ubuntu 22.04.5 LTS | mon, osd, mgr | _admin | /dev/xvdb |
| node-mon03 | 172.31.17.150  | Ubuntu 22.04.5 LTS | mon, osd, mgr | _admin | /dev/xvdb |
| node-mon04 | 172.31.24.21 | Ubuntu 22.04.5 LTS | mon, osd | | /dev/xvdb |
| node-mon05 | 172.31.17.124 | Ubuntu 22.04.5 LTS | mon, osd | | /dev/xvdb |
| node-osd01  | 172.31.25.57 | Ubuntu 22.04.5 LTS | osd | | /dev/xvdb |

## Install & pre-config on all nodes
**Check timezone & NTP Time sync**
```
# timedatectl 
               Local time: Fri 2025-09-05 02:27:16 UTC
           Universal time: Fri 2025-09-05 02:27:16 UTC
                 RTC time: Fri 2025-09-05 02:27:16
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```
```
# systemctl status chrony
● chrony.service - chrony, an NTP client/server
     Loaded: loaded (/lib/systemd/system/chrony.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-09-05 02:14:12 UTC; 14min ago
       Docs: man:chronyd(8)
             man:chronyc(1)
             man:chrony.conf(5)
   Main PID: 477 (chronyd)
      Tasks: 2 (limit: 4670)
     Memory: 2.0M
        CPU: 104ms
     CGroup: /system.slice/chrony.service
             ├─477 /usr/sbin/chronyd -F 1
             └─501 /usr/sbin/chronyd -F 1

Sep 05 02:14:12 ip-172-31-24-155 systemd[1]: Starting chrony, an NTP client/server...
Sep 05 02:14:12 ip-172-31-24-155 chronyd[477]: chronyd version 4.2 starting (+CMDMON +NTP +REFCLOCK +RTC +PRIVDROP +SCFILTER +SIGND +ASYNCDNS +NTS +SECHASH +IPV6 -DEBUG)
Sep 05 02:14:12 ip-172-31-24-155 chronyd[477]: Using right/UTC timezone to obtain leap second data
Sep 05 02:14:12 ip-172-31-24-155 chronyd[477]: Loaded seccomp filter (level 1)
Sep 05 02:14:12 ip-172-31-24-155 systemd[1]: Started chrony, an NTP client/server.
Sep 05 02:14:19 ip-172-31-24-155 chronyd[477]: Selected source 169.254.169.123
Sep 05 02:14:19 ip-172-31-24-155 chronyd[477]: System clock TAI offset set to 37 seconds
```

**Set hostname**
```
# hostnamectl set-hostname <SERVER_HOSTNAME>

Example:
# hostnamectl set-hostname node-mon01
```

**Change default values system limit**
```
# vi /etc/systemd/system.conf
---
DefaultLimitNOFILE=65000
DefaultLimitNPROC=65000
DefaultTasksMax=65000
```

**Install Podman**
```
# apt-get update && apt-get install -y podman

# systemctl status podman
○ podman.service - Podman API Service
     Loaded: loaded (/lib/systemd/system/podman.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Fri 2025-09-05 02:27:09 UTC; 1min 42s ago
TriggeredBy: ● podman.socket
       Docs: man:podman-system-service(1)
    Process: 3475 ExecStart=/usr/bin/podman $LOGGING system service (code=exited, status=0/SUCCESS)
   Main PID: 3475 (code=exited, status=0/SUCCESS)
        CPU: 89ms

Sep 05 02:27:04 node-mon01 systemd[1]: Starting Podman API Service...
Sep 05 02:27:04 node-mon01 systemd[1]: Started Podman API Service.
Sep 05 02:27:04 node-mon01 podman[3475]: time="2025-09-05T02:27:04Z" level=info msg="/usr/bin/podman filtering at log level info"
Sep 05 02:27:04 node-mon01 podman[3475]: time="2025-09-05T02:27:04Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
Sep 05 02:27:04 node-mon01 podman[3475]: 2025-09-05 02:27:04.504944434 +0000 UTC m=+0.323604301 system refresh
Sep 05 02:27:04 node-mon01 podman[3475]: time="2025-09-05T02:27:04Z" level=info msg="Setting parallel job count to 7"
Sep 05 02:27:04 node-mon01 podman[3475]: time="2025-09-05T02:27:04Z" level=info msg="using systemd socket activation to determine API endpoint"
Sep 05 02:27:04 node-mon01 podman[3475]: time="2025-09-05T02:27:04Z" level=info msg="using API endpoint: ''"
Sep 05 02:27:04 node-mon01 podman[3475]: time="2025-09-05T02:27:04Z" level=info msg="API service listening on \"/run/podman/podman.sock\""
Sep 05 02:27:09 node-mon01 systemd[1]: podman.service: Deactivated successfully.
```

**Check Python version**
```
# python3 -V
Python 3.10.12
```

**Check disk device**
```
# lsblk 
NAME     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0      7:0    0 27.6M  1 loop /snap/amazon-ssm-agent/11797
loop1      7:1    0 63.8M  1 loop /snap/core20/2599
loop2      7:2    0 73.9M  1 loop /snap/core22/2045
loop3      7:3    0 89.4M  1 loop /snap/lxd/31333
loop4      7:4    0 49.3M  1 loop /snap/snapd/24792
xvda     202:0    0   20G  0 disk 
├─xvda1  202:1    0 19.9G  0 part /
├─xvda14 202:14   0    4M  0 part 
└─xvda15 202:15   0  106M  0 part /boot/efi
xvdb     202:16   0   50G  0 disk
```

**Install cephadm**
Check active release version. In current, active release version is "squid - 19.2.3"
```
# CEPH_RELEASE="19.2.3" && curl --silent --remote-name --location https://download.ceph.com/rpm-${CEPH_RELEASE}/el9/noarch/cephadm

# chmod +x cephadm

# ./cephadm install && which cephadm 
Installing packages ['cephadm']...
/usr/sbin/cephadm
```

**Install ceph-common**
```
# cephadm add-repo --release squid
Installing repo GPG key from https://download.ceph.com/keys/release.gpg...
Installing repo file at /etc/apt/sources.list.d/ceph.list...
Updating package list...
Completed adding repo.

# cephadm install ceph-common
Installing packages ['ceph-common']...

# ceph --version
ceph version 19.2.3 (c92aebb279828e9c3c1f5d24613efca272649e62) squid (stable)
```

## Bootstrap a new cluster (Perform in node-mon01)
```
# cephadm bootstrap --mon-ip <IP server node-mon01>

Example:
# cephadm bootstrap --mon-ip 172.31.24.155
Verifying podman|docker is present...
Verifying lvm2 is present...
Verifying time synchronization is in place...
Unit chrony.service is enabled and running
Repeating the final host check...
podman (/usr/bin/podman) version 3.4.4 is present
systemctl is present
lvcreate is present
Unit chrony.service is enabled and running
Host looks OK
Cluster fsid: b0c8c6be-8a07-11f0-8f49-7b896d8c3aba
Verifying IP 172.31.24.155 port 3300 ...
Verifying IP 172.31.24.155 port 6789 ...
Mon IP `172.31.24.155` is in CIDR network `172.31.16.0/20`
Mon IP `172.31.24.155` is in CIDR network `172.31.16.0/20`
...
...
Waiting for mgr epoch 9...
mgr epoch 9 is available
Generating a dashboard self-signed certificate...
Creating initial admin user...
Fetching dashboard port number...
Ceph Dashboard is now available at:

	     URL: https://ip-172-31-24-155.ap-southeast-1.compute.internal:8443/
	    User: admin
	Password: uywa2hmick

Enabling client.admin keyring and conf on hosts with "admin" label
Saving cluster configuration to /var/lib/ceph/b0c8c6be-8a07-11f0-8f49-7b896d8c3aba/config directory
Enabling autotune for osd_memory_target
You can access the Ceph CLI as following in case of multi-cluster or non-default config:

	sudo /usr/sbin/cephadm shell --fsid b0c8c6be-8a07-11f0-8f49-7b896d8c3aba -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

Or, if you are only running a single cluster on this host:

	sudo /usr/sbin/cephadm shell 

Please consider enabling telemetry to help improve Ceph:

	ceph telemetry on

For more information see:

	https://docs.ceph.com/docs/master/mgr/telemetry/

Bootstrap complete.
```
