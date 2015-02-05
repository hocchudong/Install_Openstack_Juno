Skip to content
 This repository
Explore
Gist
Blog
Help
Trần Mạnh Huy huytm
 
1  Unwatch 
  Star 0
 Fork 0huytm/Oenstack_juno
Oenstack_juno/   or cancel
 Edit file   Preview changes
Oenstack_juno
Huong dan

Controller
Controller Network: eth0: 10.10.10.4610.200, 
		   network eth1: 172.16.69.4669.200 (vi/etc/network/interfaces)

add repo
```sh
# apt-get install ubuntu-cloud-keyring
# echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/juno main" > /etc/apt/sources.list.d/cloudarchive-juno.list
```
  
update va dist-upgrade

`# apt-get update -y && apt-get dist-upgrade -y`

Cai dat ntp

`apt-get install ntp`

Cau hinh ntp `vi /etc/ntp.conf`

sua dong `server ntp.ubuntu.com`

thanh 

```sh
server 0.vn.pool.ntp.org iburst 
server 1.asia.pool.ntp.org iburst 
server 2.asia.pool.ntp.org iburst
```

va 

```sh
restrict -6 default kod notrap nomodify nopeer noquery
restrict -4 default kod notrap nomodify nopeer noquery
```

thanh

```sh
restrict -4 default kod notrap nomodify
restrict -6 default kod notrap nomodify
```

restart ntp 

`service ntp restart`

Cai dat rabbitmq 

`apt-get install rabbitmq-server`

Dat pass cho rabbit
```sh
# rabbitmqctl change_password guest osjuno123a@
Changing password for user "guest" ...
...done.
```

Cai dat mysql

# apt-get install mariadb-server python-mysqldb

Tai buoc http://imgur.com/yZb0eDy

Nhap mat khau

Cau hinh file /etc/mysql/my.cnf

```sh
[mysqld]
bind-address = 0.0.0.0
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
```

bao mat cho mysql service

# mysql_secure_installation

```sh
Enter current password for root (enter for none): [your password]
Change the root password? [Y/n]: n
Remove anonymous users? [Y/n]: y
Disallow root login remotely? [Y/n]: y
Remove test database and access to it? [Y/n]: y
Reload privilege tables now? [Y/n]: y
```

Khoi dong lai dich vu

# service mysql restart

TAO CAC DB CHO CAC SERVICE

# mysql -uroot -p

Tao db cho keystone
```sh
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'10.10.10.200' IDENTIFIED BY 'osjuno123a@';
```
Tao db cho glance
```sh
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'10.10.10.200' IDENTIFIED BY 'osjuno123a@';
```
Tao db cho nova
```sh
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'10.10.10.200' IDENTIFIED BY 'osjuno123a@';
```
Tao db cho neutron
```sh
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'10.10.10.200' IDENTIFIED BY 'osjuno123a@';
```

------------------------------------------
CAI DAT KEYSTONE

apt-get install keystone python-keystoneclient

edit file /etc/keystone/keystone.conf
```sh
[DEFAULT]
...
admin_token=osjuno123a@
verbose = True
[database]
...
connection = mysql://keystone:osjuno123a@@10.10.10.200/keystone
[token]
...
provider = keystone.token.providers.uuid.Provider
driver = keystone.token.persistence.backends.sql.Token
```

Dong bo db keystone voi mysql database
 
`# su -s /bin/sh -c "keystone-manage db_sync" keystone`

restart keystone

`# service keystone restart`

xoa cac db mac dinh

`# rm -f /var/lib/keystone/keystone.db`

Cau hinh xoa cac token het han
```sh
# (crontab -l -u keystone 2>&1 | grep -q token_flush) || \
  echo '@hourly /usr/bin/keystone-manage token_flush >/var/log/keystone/keystone-tokenflush.log 2>&1' \
  >> /var/spool/cron/crontabs/keystone
```

Tao cac tenant admin

`# export OS_SERVICE_TOKEN=osjuno123a@` (Chu y lay o file /etc/keystone/keystone.conf)

`# export OS_SERVICE_ENDPOINT=http://10.10.10.200:35357/v2.0`

`$ keystone tenant-create --name admin --description "Admin Tenant"`

Tao user admin

`$ keystone user-create --name admin --pass osjuno123a@ --email admin_osjuno@gmail.com`

Tao role

`$ keystone role-create --name admin`

add role

`keystone user-role-add --user admin --tenant admin --role admin`

Tao tenant service cho cac dich vu

`$ keystone tenant-create --name service --description "Service Tenant"`

Tao cho cac service user

```sh
$ keystone user-create --name glance --pass osjuno123a@

$ keystone user-create --name nova --pass osjuno123a@

$ keystone user-create --name neutron --pass osjuno123a@
```

Add role cho cac user

```sh
# keystone user-role-add --user glance --tenant service --role admin

# keystone user-role-add --user nova --tenant service --role admin

# keystone user-role-add --user neutron --tenant service --role admin
```

Tao cac service entity

```sh
# keystone service-create --name keystone --type identity \
  --description "OpenStack Identity"

# keystone service-create --name glance --type image \
  --description "OpenStack Image Service"

# keystone service-create --name nova --type compute \
  --description "OpenStack Compute"

# keystone service-create --name neutron --type network \
  --description "OpenStack Networking"
```

Tao cac endpoint

keystone
```sh
$ keystone endpoint-create \
  --service-id $(keystone service-list | awk '/ identity / {print $2}') \
  --publicurl http://10.10.10.200:5000/v2.0 \
  --internalurl http://10.10.10.200:5000/v2.0 \
  --adminurl http://10.10.10.200:35357/v2.0 \
  --region regionOne
```
  
glance
```sh
$ keystone endpoint-create \
  --service-id $(keystone service-list | awk '/ image / {print $2}') \
  --publicurl http://10.10.10.200:9292 \
  --internalurl http://10.10.10.200:9292 \
  --adminurl http://10.10.10.200:9292 \
  --region regionOne
```
nova
```sh
$ keystone endpoint-create \
  --service-id $(keystone service-list | awk '/ compute / {print $2}') \
  --publicurl http://10.10.10.200:8774/v2/%\(tenant_id\)s \
  --internalurl http://10.10.10.200:8774/v2/%\(tenant_id\)s \
  --adminurl http://10.10.10.200:8774/v2/%\(tenant_id\)s \
  --region regionOne
```sh

neutron
```sh
$ keystone endpoint-create \
  --service-id $(keystone service-list | awk '/ network / {print $2}') \
  --publicurl http://10.10.10.200:9696 \
  --adminurl http://10.10.10.200:9696 \
  --internalurl http://10.10.10.200:9696 \
  --region regionOne
```

TAo file bien moi truong `vi admin-openrc.sh`

Noi dung

```sh
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=osjuno123a@
export OS_AUTH_URL=http://10.10.10.200:35357/v2.0
```

Cai dat glance

`apt-get install glance python-glanceclient -y`

Edit `vi /etc/glance/glance-api.conf`

```sh
[database]
...
connection = mysql://glance:osjuno123a@@10.10.10.200/glance

[keystone_authtoken]
...
auth_uri = http://10.10.10.200:5000/v2.0
identity_uri = http://10.10.10.200:35357
admin_tenant_name = service
admin_user = glance
admin_password = osjuno123a@

[paste_deploy]
...
flavor = keystone

[glance_store]
...
default_store = file
filesystem_store_datadir = /var/lib/glance/images/

[DEFAULT]
...
notification_driver = noop
verbose = True
```

Edit file `vi /etc/glance/glance-registry.conf`

```sh
[database]
...
connection = mysql://glance:osjuno123a@@10.10.10.200/glance

[keystone_authtoken]
...
auth_uri = http://10.10.10.200:5000/v2.0
identity_uri = http://10.10.10.200:35357
admin_tenant_name = service
admin_user = glance
admin_password = osjuno123a@

[paste_deploy]
...
flavor = keystone
[DEFAULT]
...
notification_driver = noop
verbose = True
```

Xoa cac db mac dinh `rm -f /var/lib/glance/glance.sqlite`

Dong bo db

`su -s /bin/sh -c "glance-manage db_sync" glance`

Restart

```sh
service glance-registry restart
service glance-api restart
```

Verify

Tao folder

```sh
$ mkdir /tmp/images
$ cd /tmp/images
```

download

`wget http://cdn.download.cirros-cloud.net/0.3.3/cirros-0.3.3-x86_64-disk.img`

source file bien moi truong

`source admin-openrc.sh`

upload

```sh
glance image-create --name "cirros-0.3.3-x86_64" --file cirros-0.3.3-x86_64-disk.img \
  --disk-format qcow2 --container-format bare --is-public True --progress
```
  
Cai dat NOVA
```sh
apt-get install nova-api nova-cert nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler python-novaclient
```

Cau hinh file `vi /etc/nova/nova.conf`

```sh
[database]
...
connection = mysql://nova:osjuno123a@@10.10.10.200/nova

[DEFAULT]
...
rpc_backend = rabbit
rabbit_host = 10.10.10.200
rabbit_password = osjuno123a@
auth_strategy = keystone
my_ip = 10.10.10.200
vncserver_listen = 10.10.10.200
vncserver_proxyclient_address = 10.10.10.200
verbose = True

[keystone_authtoken]
...
auth_uri = http://10.10.10.200:5000/v2.0
identity_uri = http://10.10.10.200:35357
admin_tenant_name = service
admin_user = nova
admin_password = osjuno123a@

[glance]
...
host = 10.10.10.200
```

Dong bo db

`su -s /bin/sh -c "nova-manage db sync" nova`

Xoa db mac dinh

`rm -f /var/lib/nova/nova.sqlite`

restart nova

```sh
service nova-api restart
service nova-cert restart
service nova-consoleauth restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
```

CAI DAT neutron

`apt-get install neutron-server neutron-plugin-ml2 python-neutronclient`

cau hinh networking server `vi /etc/neutron/neutron.conf`

```sh
[database]
...
connection = mysql://neutron:osjuno123a@@10.10.10.200/neutron

[DEFAULT]
...
rpc_backend = rabbit
rabbit_host = 10.10.10.200
rabbit_password = osjuno123a@
auth_strategy = keystone
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True
nova_url = http://10.10.10.200:8774/v2
nova_admin_auth_url = http://10.10.10.200:35357/v2.0
nova_region_name = regionOne
nova_admin_username = nova
nova_admin_tenant_id = 4636e2407a73422e8158bab7d7f9b53f
nova_admin_password = osjuno123a@
verbose = True

[keystone_authtoken]
...
auth_uri = http://10.10.10.200:5000/v2.0
identity_uri = http://10.10.10.200:35357
admin_tenant_name = service
admin_user = neutron
admin_password = osjuno123a@
```

Chu y tenant nova_admin_tenant_id = SERVICE_TENANT_ID dung lenh `keystone tenant-get service` de lay id

Cau hinh ml2 `vi /etc/neutron/plugins/ml2/ml2_conf.ini`

```sh
[ml2]
...
type_drivers = flat,gre
tenant_network_types = gre
mechanism_drivers = openvswitch

[ml2_type_gre]
...
tunnel_id_ranges = 1:1000

[securitygroup]
...
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
Cau hinh compute su dung network vi /etc/nova/nova.conf

[DEFAULT]
...
network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[neutron]
...
url = http://10.10.10.200:9696
auth_strategy = keystone
admin_auth_url = http://10.10.10.200:35357/v2.0
admin_tenant_name = service
admin_username = neutron
admin_password = osjuno123a@
```

Dong bo db

```sh
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade juno" neutron
```
  
restart nova

```sh
service nova-api restart
service nova-scheduler restart
service nova-conductor restart
```

restart neutron

service neutron-server restart

-----------------------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------
NETWORK NODE

add repo

```sh
# apt-get install ubuntu-cloud-keyring
# echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/juno main" > /etc/apt/sources.list.d/cloudarchive-juno.list
```

upgrade

`# apt-get update -y && apt-get dist-upgrade -y`

cai dat python-mysql

`# apt-get install python-mysqldb`

cai ntp

`# apt-get install ntp`

Cau hinh file `/vi/etc/ntp.conf`

comment lai cac dong

```sh
#server 0.ubuntu.pool.ntp.org
#server 1.ubuntu.pool.ntp.org
#server 2.ubuntu.pool.ntp.org
#server 3.ubuntu.pool.ntp.org

````

Sua dong `server ntp.ubuntu.com` thanh `server 10.10.10.200 iburst`

them cac dong sau vao file `vi /etc/sysctl.conf`

```sh
net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
```

Thuc thi ngay lap tuc file nay

`# sysctl -p`

Cai neutron

```sh
apt-get install neutron-plugin-ml2 neutron-plugin-openvswitch-agent \
  neutron-l3-agent neutron-dhcp-agent -y
```
  
Cau hinh file `vi /etc/neutron/neutron.conf`

```sh
[DEFAULT]
...
rpc_backend = rabbit
rabbit_host = 10.10.10.200
rabbit_password = osjuno123a@
auth_strategy = keystone
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
verbose = True

[keystone_authtoken]
...
auth_uri = http://10.10.10.200:5000/v2.0
identity_uri = http://10.10.10.200:35357
admin_tenant_name = service
admin_user = neutron
admin_password = osjuno123a@
Cau hinh ml2 vi /etc/neutron/plugins/ml2/ml2_conf.ini

[ml2]
...
type_drivers = flat,gre
tenant_network_types = gre
mechanism_drivers = openvswitch

[ml2_type_flat]
...
flat_networks = external

[securitygroup]
...
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

[ovs]
...
local_ip = 10.10.20.201 (Chu y: la ip cua may network theo dai vm-data)
enable_tunneling = True
bridge_mappings = external:br-ex

[agent]
...
tunnel_types = gre
```

Cau hinh l3-agent vi /etc/neutron/l3_agent.ini

```sh
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
use_namespaces = True
external_network_bridge = br-ex
verbose = True
```

Cau hinh dhcp-agent `vi /etc/neutron/dhcp_agent.ini`

```sh
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
use_namespaces = True
verbose = True
```

Cau hinh metadata agent `vi /etc/neutron/metadata_agent.ini`

```sh
[DEFAULT]
...
auth_url = http://10.10.10.200:5000/v2.0
auth_region = regionOne
admin_tenant_name = service
admin_user = neutron
admin_password = osjuno123a@
nova_metadata_ip = 10.10.10.200
metadata_proxy_shared_secret = osjuno123a@
verbose = True
neutron
```

Tai CONTROLLER `vi /etc/nova/nova.conf`

```sh
[neutron]
...
service_metadata_proxy = True
metadata_proxy_shared_secret = osjuno123a@
restart nova
```

restart nova

`service nova-api restart`

Cau hinh OVS service

restart OVS

`service openvswitch-switch restart`

Tao external-br

```sh
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex eth1
```

-> mat ket noi

`ethtool -K eth1 gro off`

Cau hinh lai file `vi /etc/network/interfaces`

```sh
auto eth0
iface eth0 inet static
address 10.10.10.201
netmask 255.255.255.0
network 10.10.10.0

auto eth1
iface eth1 inet manual
up ifconfig $IFACE 0.0.0.0 up
up ip link set $IFACE promisc on
down ip link set $IFACE promisc off
down ifconfig $IFACE down

auto br-ex
iface br-ex inet static
address 172.16.69.201
netmask 255.255.255.0
network 172.16.69.0
gateway 172.16.69.1
dns-nameserver 8.8.8.8

auto eth2
iface eth2 inet static
address 10.10.20.201
netmask 255.255.255.0
network 10.10.20.0
restart mang
```

restart network

`ifdown -a && ifup -a`

restart neutron

```sh
service neutron-plugin-openvswitch-agent restart
service neutron-l3-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------
COMPUTE

add repo

```sh
# apt-get install ubuntu-cloud-keyring
# echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/juno main" > /etc/apt/sources.list.d/cloudarchive-juno.list
```

update

`# apt-get update -y && apt-get dist-upgrade -y`

cai dat python-mysql

`# apt-get install python-mysqldb`

Cai nova

`apt-get install nova-compute sysfsutils`

Edit file `vi /etc/nova/nova.conf`

```sh
[DEFAULT]
...
rpc_backend = rabbit
rabbit_host = 10.10.10.200
rabbit_password = osjuno123a@
auth_strategy = keystone
my_ip = 10.10.10.202 (dia chi ip compute theo dai managet)
vnc_enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = 10.10.10.202
novncproxy_base_url = http://10.10.10.200:6080/vnc_auto.html
verbose = True

[keystone_authtoken]
...
auth_uri = http://10.10.10.200:5000/v2.0
identity_uri = http://10.10.10.200:35357
admin_tenant_name = service
admin_user = nova
admin_password = osjuno123a@

[glance]
...
host = 10.10.10.200
```

Cau hinh qemu

`egrep -c '(vmx|svm)' /proc/cpuinfo`

Neu tra ve gia tri >1 tuc la ho tro

Neu tra ve 0 thi cau hinh lai file `vi /etc/nova/nova-compute.conf`

```sh
[libvirt]
...
virt_type = qemu
```

restart nova

`service nova-compute restart`

xoa db mac dinh

`rm -f /var/lib/nova/nova.sqlite`


Cai dat neutron

them cac dong sau vao file `vi /etc/sysctl.conf`

```sh
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
sysctl -p
```

Cai neutron

`apt-get install neutron-plugin-ml2 neutron-plugin-openvswitch-agent`

Edit file `vi /etc/neutron/neutron.conf`

```sh
[DEFAULT]
...
rpc_backend = rabbit
rabbit_host = 10.10.10.200
rabbit_password = osjuno123a@
auth_strategy = keystone
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
verbose = True

[keystone_authtoken]
...
auth_uri = http://10.10.10.200:5000/v2.0
identity_uri = http://10.10.10.200:35357
admin_tenant_name = service
admin_user = neutron
admin_password = osjuno123a@
```

Cau hinh ml2 `vi /etc/neutron/plugins/ml2/ml2_conf.ini`

```sh
[ml2]
...
type_drivers = flat,gre
tenant_network_types = gre
mechanism_drivers = openvswitch

[ml2_type_gre]
...
tunnel_id_ranges = 1:1000

[securitygroup]
...
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

[ovs]
...
local_ip = 10.10.20.202
enable_tunneling = True

[agent]
...
tunnel_types = gre
```

cau hinh OVS `service openvswitch-switch restart`

Cau hinh compute su dung networking `vi /etc/nova/nova.conf`

```sh
[DEFAULT]
...
network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[neutron]
...
url = http://10.10.10.200:9696
auth_strategy = keystone
admin_auth_url = http://10.10.10.200:35357/v2.0
admin_tenant_name = service
admin_username = neutron
admin_password = osjuno123a@
```

restart nova

`service nova-compute restart`

restart ovs

`service neutron-plugin-openvswitch-agent restart`

Trần Mạnh Huy
Commit changes


  Commit directly to the master branch
  Create a new branch for this commit and start a pull request. Learn more about pull requests.
Commit changes  Cancel
Status API Training Shop Blog About
© 2015 GitHub, Inc. Terms Privacy Security Contact
