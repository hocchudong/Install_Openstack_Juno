# Oenstack_juno
Huong dan

Controller
Network: eth0: 10.10.10.46, network eth1: 172.16.69.46
(vi/etc/network/interfaces)

Update

`# apt-get update -y`

reboot

`# itnit 6`


`# apt-get install ubuntu-cloud-keyring`

`# echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/juno main" > /etc/apt/sources.list.d/cloudarchive-juno.list`

`# apt-get update -y && apt-get dist-upgrade -y`

`# apt-get install mariadb-server python-mysqldb`

```sh
[mysqld]
bind-address = 10.0.0.11
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
```

