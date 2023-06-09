# DLP_Project

This is a guide to setup the full DLP project.

## Project Description

- Three virtual servers
- Web services: Apache and PHP
- Database: MariaDB
- Web files distributed on two servers using the DRBD system
- Database service operating on a Galera Cluster system across all three servers
- Communication between the web server and the database server through haproxy
- The web server should connect to the local server (localhost)
- In case of insufficient capacity, the web server should connect to another database server
- The web service should include a simple page that connects to the database or any ready-made system.

## Prerequisites

### VMWare

You need `Vmware Workstation Pro` in our case we are using vmware 7

### AlmaLinux

We are using `AlmaLinux-9-minimal` iso as requested by the college

## Setting up the VMs

### Creating the VMs

- Create 3 VMs with the following specs:
  - using the `AlmaLinux-9-minimal` iso
  - 2 vCPUs
  - 2 GB RAM
  - 10 GB HDD [or more]

### Network configuration

in my case I have the following ip addresses:

- node01 192.168.110.130
- node02 192.168.110.131
- node03 192.168.110.132

### Hostname configuration

```bash
# change the hostname on each vm
 hostnamectl set-hostname node01
```

### Define Hostnames

```bash
vi /etc/hosts
```

```conf
# append the following lines
192.168.110.130 node01.khader.com node01
192.168.110.131 node02.khader.com node02
192.168.110.132 node03.khader.com node03
```

### DRBD Setup

We need to install the DRBD on the machines

```bash
yum install https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum install -y kmod-drbd9x drbd9x-utils
```

Open DRBD port on the firewall

```bash
sudo firewall-cmd --add-port=6996-7800/tcp --permanent
sudo firewall-cmd --reload
```

finally we need to add the new resource to the `/etc/drbd.d/resource0.res` file

```bash
vi /etc/drbd.d/resource0.res
```

```conf
resource resource0 {

  on node01 {

  device /dev/drbd1;

  disk /dev/sda;

  address 192.168.110.130:7789;

  meta-disk internal;

  }

  on node02 {
  device /dev/drbd1;

  disk /dev/sda;

  address 192.168.110.131:7789;

  meta-disk internal;

  }
}
```

Enable the DRBD service

```bash
sudo drbdadm create-md resource0
sudo drbdadm up resource0
systemctl enable drbd --now
```

Mount the DRBD partition

```bash
sudo drbdadm primary --force resource0
mkfs.ext4 /dev/drbd1
mkdir /mnt/drbd
mount /dev/drbd1 /mnt/drbd
```

### Web & DB Software Setup
Open the following ports on the firewall

```bash
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=3306/tcp --permanent
sudo firewall-cmd --add-port=4567/tcp --permanent
sudo firewall-cmd --add-port=4568/tcp --permanent
sudo firewall-cmd --add-port=4444/tcp --permanent
sudo firewall-cmd --add-port=4567/udp --permanent
sudo firewall-cmd --add-port=4568/udp --permanent
sudo firewall-cmd --add-port=4444/udp --permanent
sudo firewall-cmd --reload
```

```bash
# mariadb
yum install mariadb-server -y
systemctl enable mariadb --now

# apache
yum install php httpd -y
systemctl enable httpd --now
```

### Galera Cluster Setup

```bash
yum install mariadb-server -y
yum install mariadb-server-galera -y
```

Then setup mysql credentials

```bash
mysql_secure_installation
```

update `/etc/my.cnf.d/mariadb-server.cnf`

```conf
vi /etc/my.cnf.d/mariadb-server.cnf
```

```conf
[galera]
# Mandatory settings
wsrep_on=ON
#wsrep_provider=
wsrep_cluster_address="gcomm://192.168.110.130,192.168.110.131,192.168.110.132"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
# Allow server to accept connections on all interfaces.
bind-address=0.0.0.0
wsrep_cluster_name="cluster1"
wsrep_sst_method=rsync
wsrep_node_address="192.168.110.130"
wsrep_node_name="node01"
```
