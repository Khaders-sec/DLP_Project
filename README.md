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

