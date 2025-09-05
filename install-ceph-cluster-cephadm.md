# Deploy Ceph cluster with cephadm

| Requirements |
| :--- |
| Python 3 |
| Systemd |
| Podman or Docker for running containers |
| Time synchronization (such as Chrony or the legacy ntpd) |
| LVM2 for provisioning storage devices |


## Lab info (3 master, 3 worker)
| Hostname | IP Address | OS | Role | Label |
| :--- | :--- | :--- | :--- | :--- |
| node-mon01 |  | Ubuntu 22.04.5 LTS | mon, osd, mgr | _admin |
| node-mon02 |  | Ubuntu 22.04.5 LTS | mon, osd, mgr | _admin |
| node-mon03 |  | Ubuntu 22.04.5 LTS | mon, osd, mgr | _admin |
| node-mon04 |  | Ubuntu 22.04.5 LTS | mon, osd | |
| node-mon05 |  | Ubuntu 22.04.5 LTS | mon, osd | |
| node-osd01  |  | Ubuntu 22.04.5 LTS | osd | |
