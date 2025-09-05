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
| node-mon02 |  | Ubuntu 22.04.5 LTS | mon, osd, mgr | _admin | /dev/xvdb |
| node-mon03 |  | Ubuntu 22.04.5 LTS | mon, osd, mgr | _admin | /dev/xvdb |
| node-mon04 |  | Ubuntu 22.04.5 LTS | mon, osd | | /dev/xvdb |
| node-mon05 |  | Ubuntu 22.04.5 LTS | mon, osd | | /dev/xvdb |
| node-osd01  |  | Ubuntu 22.04.5 LTS | osd | | /dev/xvdb |

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
```
