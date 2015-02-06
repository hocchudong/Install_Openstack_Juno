#HƯỚNG DẪN CÀI ĐẶT OPENSTACK-JUNO TRÊN UBUNTU 14.04.1

Bài viết này tham khảo tại địa chỉ http://docs.openstack.org/juno/install-guide/install/apt/content/

*Các mật khẩu trong quá trình cài đặt đều là :* **osjuno123a@** 
----------------------------------------------------------------------------------------------------------------------------
[A. Mô hình cài đặt](#A)

[B. Các bước cài đặt chính](#B)
- [I. CONTROLLER NODE](#I)
<ul>
<li>    	[1.1. Enable OpenStack repository](#ctl)</li>
<li>    	[1.2 Upgrade các packages hệ thống](#12)</li>
<li>    	[1.3. Cài đặt NTP service.](#13)</li>
<li>    	[1.4. Cài đặt RabbitMQ làm Messaging server.](#14)</li>
<li>		[1.5. Cài đặt Database server.](#1.5)</li>
<li>		[1.6 Cài đặt Keystone.](#1.6)</li>
<li>		[1.7. Tạo các Tenant, User, Role.](#1.7)</li>
<li>		[1.8 Tạo Script biến môi trường](#1.8)</li>
<li>		[1.9. Cài đặt Glance.](#1.9)</li>
<li>		[1.10 Cài đặt Nova.](#1.10)</li>
<li>		[1.11 Cài đặt Neutron.](#1.11)</li>
</ul>
- [II. NETWORK NODE](#II)
<ul>
<li>	[2.1. Enable OpenStack repository.](#2.1)</li>
<li>	[2.2. Upgrade các packages hệ thống.](#2.2)</li>
<li>	[2.3. Cài đặt python-mysql.](#2.3)</li>
<li>	[2.4. Cài đặt NTP service.](#2.4)</li>
<li>	[2.5 Cài đặt Neutron.](#2.5)</li>
<li>	[2.6 Cấu hình OVS service..](#2.6)</li>
</ul>
- [III. COMPUTE NODE](#III)
<ul>
<li>		[3.1. Enable OpenStack repository.](#3.1)</li>
<li>		[3.2. Upgrade các packages hệ thống.](#3.2)</li>
<li>		[3.3. Cài đặt python-mysql.](#3.3)</li>
<li>		[3.4. Cài đặt NTP service.](#3.4)</li>
<li>		[3.5 Cài đặt Nova.](#3.5)</li>
<li>		[3.6 Cài đặt Neutron.](#3.6)</li>
</ul>
- [IV. Cài đặt Horizon](#IV)

[C. Kiểm tra lại quá trình cài đặt](#C)
<ul>
<li>		 [a. Kiểm tra hoạt động của các dịch vụ.]</li>(#a)
<li>		 [b. Tạo external network.]</li>(#b)
<li>	 	 [c. Tạo internal network.]</li>(#c) 
<li>		 [d. Tạo router.]</li>(#d)
<li>		 [e. Kiểm tra lại mô hình mạng.]</li>(#e)
<li>		 [f. Khởi tạo 1 instances.]</li>(#f)
</ul>
[D. Tài liệu tham khảo](#D)

----------------------------------------------------------------------------------------------------------------------------
<a name="A"></a>
#A. Mô hình cài đặt 

<img src="http://i.imgur.com/qzMwJ62.png">

<ul><ul><ul><ul><ul><ul><ul><ul><ul>Mô hình cài đặt Openstack Juno theo 3 node</ul></ul></ul></ul></ul></ul></ul></ul></ul>

<a name="B"></a>

#B. Các bước cài đặt chính
<a name="I"></a>

##I. CONTROLLER NODE

<a name="ctl"></a>
####1.1 Enable OpenStack repository.
```sh
# apt-get install ubuntu-cloud-keyring
# echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/juno main" > /etc/apt/sources.list.d/cloudarchive-juno.list
```

<a name="12"></a>  
#####1.2 Upgrade các packages hệ thống.

`# apt-get update -y && apt-get dist-upgrade -y`

<a name="13"></a>
#####1.3.	Cài đặt NTP service.
<ul>
NTP (Network Time protocol) sử dụng để đồng bộ hóa về thời gian giữa các dịch vụ ở các node khác nhau
</ul>

- *Download và cài đặt NTP service* 

`# apt-get install ntp -y`

- *Cấu hình NTP service tại file `vi /etc/ntp.conf`*

- *Sửa dòng* `server ntp.ubuntu.com`
<ul>
*thành*
</ul>
```sh
server 0.vn.pool.ntp.org iburst 
server 1.asia.pool.ntp.org iburst 
server 2.asia.pool.ntp.org iburst
```


- *Và sửa đoạn*
<ul>
```sh
restrict -6 default kod notrap nomodify nopeer noquery
restrict -4 default kod notrap nomodify nopeer noquery
```
</ul>

<ul>
*thành*
</ul>
```sh
restrict -4 default kod notrap nomodify
restrict -6 default kod notrap nomodify
```


- *restart ntp* 

`# service ntp restart`

<a name="14"></a>
#####1.4.	Cài đặt RabbitMQ làm Messaging server.
<ul>
Openstack sử dụng một messaging server để thông báo thông tin trạng thái, và sự phối hợp các dịch vụ trong hệ thống
</ul>
- *Download và cài đặt rabbitqm là massaging server* 

`# apt-get install rabbitmq-server -y`

- *Đặt lại mật khẩu cho Rabbitmq*
```sh
# rabbitmqctl change_password guest osjuno123a@
Changing password for user "guest" ...
...done.
```

<a name="1.5"></a>
#####1.5.	Cài đặt Database server.
<ul>
Hầu hết các dịch vụ của Openstack cần phải có một hệ quản trị cơ sở dữ liệu để lưu trữ thông tin. Ở đây ta sẽ cài đặt mariadb-server làm database server chính để lưu trữ thông tin của các server.( Ngoài ra bạn cũng có thể sử dụng mysql-server để thay thế cho mariadb-server, tùy vào hoàn cành sử dụng).
</ul>
- *Download và cài đặt mariadb làm database server*

`# apt-get install mariadb-server python-mysqldb -y`

- Tại bước 

<img src="http://tecadmin.net/wp-content/uploads/2014/12/mariadb-10-ubuntu.png">

*Nguồn techadmin.net*

=> Nhập mật khẩu cho Database server



- *Cấu hình file /etc/mysql/my.cnf*
<ul>
Cấu hình để cho phép các node khác trong mạng có thể truy cập đến Database server được cài đặt trên controller node, và cấu hình các tùy chọn hữu ích.
</ul>
```sh
[mysqld]
bind-address = 0.0.0.0
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
```

- *Secure database service*

`# mysql_secure_installation`

- *Thực hiện tuần tự theo các bước sau*
```sh
Enter current password for root (enter for none): [your password]
Change the root password? [Y/n]: n
Remove anonymous users? [Y/n]: y
Disallow root login remotely? [Y/n]: y
Remove test database and access to it? [Y/n]: y
Reload privilege tables now? [Y/n]: y
```

- *Khởi động lại dịch vụ*

`# service mysql restart`

- *Tạo các database cho từng service chính của Openstack**
<ul>
Các project chính của Openstack gồm có **Keystone**, **Glance**, **Nova**, **Neutron**. Mỗi một service này cần một database riêng để lưu trữ cơ sở dữ liệu của mình.
</ul>
<ul>
Để tạo các database này ta thực hiện theo bước sau:
</ul>
`# mysql -uroot -p`

*Tạo database cho Keystone*
```sh
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'10.10.10.200' IDENTIFIED BY 'osjuno123a@';
```
*Tạo database cho Glance*
```sh
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'10.10.10.200' IDENTIFIED BY 'osjuno123a@';
```
*Tạo database cho Nova*
```sh
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'10.10.10.200' IDENTIFIED BY 'osjuno123a@';
```
*Tạo database cho Neutron*
```sh
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'osjuno123a@';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'10.10.10.200' IDENTIFIED BY 'osjuno123a@';
```

<a name="1.6"></a>
####1.6 Cài đặt Keystone

- *Cài đặt keystone để làm Identity service cho Openstack*

`# apt-get install keystone python-keystoneclient -y`

- *Cấu hình chính keystone tại file* `vi /etc/keystone/keystone.conf`
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

- *Xóa database mặc định khi cài đặt keystone*

`# rm -f /var/lib/keystone/keystone.db`

- *Đồng bộ database của keystone*
 
`# su -s /bin/sh -c "keystone-manage db_sync" keystone`

- *Restart keystone*

`# service keystone restart`

- *Cấu hình để xóa các token hết hạn*
<ul>
Theo mặc định, Identity service sẽ vẫn lưu các token hết hạn trong database, điều này làm tăng kích thước của database và giảm hiệu suất dịch vụ. Để cấu hình xóa các token hết hạn này theo 1 giờ ta thực hiện câu lệnh sau:
</ul>
```sh
# (crontab -l -u keystone 2>&1 | grep -q token_flush) || \
  echo '@hourly /usr/bin/keystone-manage token_flush >/var/log/keystone/keystone-tokenflush.log 2>&1' \
  >> /var/spool/cron/crontabs/keystone
```

<a name="1.7"></a>
####1.7.  Tạo các Tenant, User, Role.
<ul>
Dịch vụ xác thực (Identity service) trong Open stack ngoài nhiệm vụ xác thực người dùng, còn được dùng để xác thực và ủy quyền giữa các dịch vụ với nhau trong Openstack. Để tạo các user, service entity, service endpoint tương ứng cho từng dịch vụ ta thực hiện tuần tự theo các bước sau:
</ul>

- *Tạo tenant admin*

`# export OS_SERVICE_TOKEN=osjuno123a@` (**Chú ý** OS_SERVICE_TOKEN phải giống với admin_token ở file /etc/keystone/keystone.conf)

`# export OS_SERVICE_ENDPOINT=http://10.10.10.200:35357/v2.0`

`# keystone tenant-create --name admin --description "Admin Tenant"`

- *Tạo user admin*

`# keystone user-create --name admin --pass osjuno123a@ --email admin_osjuno@gmail.com`

- *Tao role admin*

`# keystone role-create --name admin`

- *Add role vừa khởi tạo cho user admin và tenant admin*

`# keystone user-role-add --user admin --tenant admin --role admin`

- *Tao tenant service cho các dịch vụ*

`# keystone tenant-create --name service --description "Service Tenant"`

- *Tao cho các service user*

```sh
# keystone user-create --name glance --pass osjuno123a@

# keystone user-create --name nova --pass osjuno123a@

# keystone user-create --name neutron --pass osjuno123a@
```

- *Add role cho các user*

```sh
# keystone user-role-add --user glance --tenant service --role admin

# keystone user-role-add --user nova --tenant service --role admin

# keystone user-role-add --user neutron --tenant service --role admin
```

- *Tạo các service entity cho từng dịch vụ*

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

- *Tạo các endpoint tương ứng cho từng dịch vụ*

*keystone*
```sh
# keystone endpoint-create \
  --service-id $(keystone service-list | awk '/ identity / {print $2}') \
  --publicurl http://10.10.10.200:5000/v2.0 \
  --internalurl http://10.10.10.200:5000/v2.0 \
  --adminurl http://10.10.10.200:35357/v2.0 \
  --region regionOne
```
  
*glance*
```sh
# keystone endpoint-create \
  --service-id $(keystone service-list | awk '/ image / {print $2}') \
  --publicurl http://10.10.10.200:9292 \
  --internalurl http://10.10.10.200:9292 \
  --adminurl http://10.10.10.200:9292 \
  --region regionOne
```

*nova*
```sh
# keystone endpoint-create \
  --service-id $(keystone service-list | awk '/ compute / {print $2}') \
  --publicurl http://10.10.10.200:8774/v2/%\(tenant_id\)s \
  --internalurl http://10.10.10.200:8774/v2/%\(tenant_id\)s \
  --adminurl http://10.10.10.200:8774/v2/%\(tenant_id\)s \
  --region regionOne
```

*neutron*
```sh
# keystone endpoint-create \
  --service-id $(keystone service-list | awk '/ network / {print $2}') \
  --publicurl http://10.10.10.200:9696 \
  --adminurl http://10.10.10.200:9696 \
  --internalurl http://10.10.10.200:9696 \
  --region regionOne
```

<a name="1.8"></a>
####1.8 Tạo Script biến môi trường
<ul>
Tạo script biến môi trường cho admin user để tải các thông tin thích hợp cho các hoạt động cấu hình sau này
</ul>

- *Tạo file biến môi trườnng* `vi admin-openrc.sh`

- *Với nội dung*

```sh
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=osjuno123a@
export OS_AUTH_URL=http://10.10.10.200:35357/v2.0
```

<a name="1.9"></a>
####1.9.	Cài đặt Glance

- *Download và cài đặt glance làm Image service cho Openstack*

`# apt-get install glance python-glanceclient -y`

- *Cấu hình chính Glance* `# vi /etc/glance/glance-api.conf`
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

- *Edit file* `vi /etc/glance/glance-registry.conf`
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

- *Xóa database mặc định của Glance* `rm -f /var/lib/glance/glance.sqlite`

- *Đồng bộ database của Glance với server*

`# su -s /bin/sh -c "glance-manage db_sync" glance`

- *Restart Glance*

```sh
service glance-registry restart
service glance-api restart
```

<a name="1.10"></a>
####1.10 Cài đặt Nova

- *Download và cài đặt nova làm compute service*
```sh
# apt-get install nova-api nova-cert nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler python-novaclient -y
```

- *Cấu hình Nova* `vi /etc/nova/nova.conf`

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

- *Xóa database mặc định của Nova*

`# rm -f /var/lib/nova/nova.sqlite`

- *Đồng bộ Nova database với server*

`# su -s /bin/sh -c "nova-manage db sync" nova`

- *Restart nova*

```sh
service nova-api restart
service nova-cert restart
service nova-consoleauth restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
```

<a name="1.11"></a>
####1.11 Cài đặt Neutron

- *Download và cài đặt Neutron*

`# apt-get install neutron-server neutron-plugin-ml2 python-neutronclient -y`

- *Cấu hình networking server* `vi /etc/neutron/neutron.conf`

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

**Chú ý**  nova_admin_tenant_id = , Ta dùng lệnh `keystone tenant-get service` để lấy id

- *Cấu hình ML2 plug-in* `vi /etc/neutron/plugins/ml2/ml2_conf.ini`

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
```

- *Cau hinh compute su dung network* vi /etc/nova/nova.conf
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

- *Đồng bộ neutron database*

```sh
# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade juno" neutron
```
  
- *Restart Nova*

```sh
service nova-api restart
service nova-scheduler restart
service nova-conductor restart
```

- *Restart neutron*

`# service neutron-server restart``

----------------------------------------------------------------------------------------------------------------------------

<a name="II"></a>
##II. NETWORK NODE

<a name="2.1"></a>
####2.1.	Enable OpenStack repository.

```sh
# apt-get install ubuntu-cloud-keyring
# echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/juno main" > /etc/apt/sources.list.d/cloudarchive-juno.list
```

<a name="2.2"></a>
####2.2.	Upgrade các packages hệ thống.

`# apt-get update -y && apt-get dist-upgrade -y`

<a name="2.3"></a>
####2.3.	Cài đặt python-mysql.

`# apt-get install python-mysqldb -y`

<a name="2.4"></a>
####2.4.	Cài đặt NTP service

- *Download và cài đặt NTP*

`# apt-get install ntp -y`

- *Cấu hình NTP* `vi /etc/ntp.conf`
<ul>
comment lại các dòng sau
</ul>
```sh
#server 0.ubuntu.pool.ntp.org
#server 1.ubuntu.pool.ntp.org
#server 2.ubuntu.pool.ntp.org
#server 3.ubuntu.pool.ntp.org

````
<ul>
Sửa dòng `server ntp.ubuntu.com` thàhh `server 10.10.10.200 iburst`
</ul>

<a name="2.5"></a>
####2.5 Cài đặt Neutron

- *Cấu hình các thông số mạng*  `vi /etc/sysctl.conf` *và thêm nội dung sau*
```sh
net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
```

- *Thực hiện các thay đổi thông số mạng này ngay lập tức:*

`# sysctl -p`

- *Download và cài đặt Neutron*
```sh
# apt-get install neutron-plugin-ml2 neutron-plugin-openvswitch-agent \
  neutron-l3-agent neutron-dhcp-agent -y
```
  
- *Cấu hình neutron* `vi /etc/neutron/neutron.conf`

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
local_ip = 10.10.20.201 #(Chu y: la ip cua may network theo dai vm-data)
enable_tunneling = True
bridge_mappings = external:br-ex

[agent]
...
tunnel_types = gre
```

- *Cấu hình l3-agent* `vi /etc/neutron/l3_agent.ini`

```sh
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
use_namespaces = True
external_network_bridge = br-ex
verbose = True
```

- *Cấu hình dhcp-agent* `vi /etc/neutron/dhcp_agent.ini`

```sh
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
use_namespaces = True
verbose = True
```

- *Cấu hình metadata agent* `vi /etc/neutron/metadata_agent.ini`

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

- *QUAY LẠI CONTROLLER NODE* `vi /etc/nova/nova.conf`

```sh
[neutron]
...
service_metadata_proxy = True
metadata_proxy_shared_secret = osjuno123a@
restart nova
```

- *Restart nova trên controller node*

`# service nova-api restart`

<a name="2.6"></a>
####2.6 Cấu hình OVS service
<ul>
OVS giúp tạo ra mạng ảo cho các instance. br-int xử lý traffic bên trong, br-ex xử lý các traffic bên ngoài. Vì thế br-ex cần dùng một interface vật lý bên ngoài để cho phép các truy cập mạng bên ngoài. Về bản chất, cổng này kết nối mạng ảo với mạng vật lý bên ngoài
</ul>

- *Restart OVS*

`# service openvswitch-switch restart`

- *Tạo external-br*

```sh
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex eth1
```

<ul>
**Chú ý:** Sau khi thực hiện bước tạo external-br và add port br-ex vào eth1 sẽ bị mất kết nối ssh đến network node, để thực hiện các cấu hình tiếp theo ta có nhiều cách để truy cập vào network node. Có thể cấu hình trực tiếp trên máy ảo VMware, hoặc từ máy controller ta sử dụng câu lệnh `ssh uvdc@10.10.10.201`sau đó nhập password để truy cập vào máy network với username là uvdc theo dải mạng management nối giữa các máy
</ul>

- *Tạm thời vô hiệu hóa GRO trên external interface*

`# ethtool -K eth1 gro off`

- *Cấu hình lại mạng cho network node* `vi /etc/network/interfaces`

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

- *Restart network*

`# ifdown -a && ifup -a`

- *Restart neutron*

```sh
service neutron-plugin-openvswitch-agent restart
service neutron-l3-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart
```
----------------------------------------------------------------------------------------------------------------------------

<a name="III"></a>
##III COMPUTE NODE

<a name="3.1"></a>
####3.1.	Enable OpenStack repository.

```sh
# apt-get install ubuntu-cloud-keyring
# echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/juno main" > /etc/apt/sources.list.d/cloudarchive-juno.list
```

<a name="3.2"></a>
####3.2.	Upgrade các packages hệ thống.

`# apt-get update -y && apt-get dist-upgrade -y`

<a name="3.3"></a>
####3.3.	Cài đặt python-mysql.

`# apt-get install python-mysqldb -y`

<a name="3.4"></a>
####3.4.	Cài đặt NTP service

- *Download và cài đặt NTP*

`# apt-get install ntp -y`

- *Cấu hình NTP* `vi /etc/ntp.conf`
<ul>
comment lại các dòng sau
</ul>
```sh
#server 0.ubuntu.pool.ntp.org
#server 1.ubuntu.pool.ntp.org
#server 2.ubuntu.pool.ntp.org
#server 3.ubuntu.pool.ntp.org

````
<ul>
Sửa dòng `server ntp.ubuntu.com` thàhh `server 10.10.10.200 iburst`
</ul>

<a name="3.5"></a>
####3.5 Cài đặt Nova

- *Download và cài đặt nova*
`# apt-get install nova-compute sysfsutils -y`

- *Cấu hình nova* `vi /etc/nova/nova.conf`

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

- *Xác định compute node hỗ trợ tăng tốc phần cứng máy ảo nào*

`# egrep -c '(vmx|svm)' /proc/cpuinfo`

Nếu trả về giá trị >=1 thì compute node hỗ trợ tăng tốc phần cứng, không cần cấu hình thêm

Nếu trả về giá trị = 0 ta cần cấu hình lại `vi /etc/nova/nova-compute.conf`

```sh
[libvirt]
...
virt_type = qemu
```

- *Restart nova*

`# service nova-compute restart`

- *Xóa nova database mặc định*

`# rm -f /var/lib/nova/nova.sqlite`

<a name="3.6"></a>
####3.6 Cài đặt Neutron

*Cấu hình các thông số mạng*  `vi /etc/sysctl.conf` *và thêm nội dung sau*

```sh
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
```

- *Thực hiện các thay đổi thông số mạng này ngay lập tức:*

`# sysctl -p`

- *Download và cài đặt neutron*

`# apt-get install neutron-plugin-ml2 neutron-plugin-openvswitch-agent -y`

- *Cấu hình neutron trên computenode* `vi /etc/neutron/neutron.conf`

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

- *Cấu hình ml2-plugin* `vi /etc/neutron/plugins/ml2/ml2_conf.ini`

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

- *Cấu hình OVS* 

`# service openvswitch-switch restart`

- Cấu hình compute sử dụng networking `vi /etc/nova/nova.conf`

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

- *Restart nova*

`# service nova-compute restart`

- *Restart OVS*

`# service neutron-plugin-openvswitch-agent restart`

<a name="IV"></a>
##IV. Cài đặt Horizon
<ul>
Sau khi thực hiện xong các bước cài đặt ở trên, ta quay lại controller node và thực hiện cài đặt horizon-dashboard
</ul>

- *Cài đặt openstack-dashboard*

`# apt-get install openstack-dashboard apache2 libapache2-mod-wsgi memcached python-memcache -y`

- *Xóa theme mặc định Ubuntu*

`# dpkg --purge openstack-dashboard-ubuntu-theme`

- *Fix bug apache2*
```sh
#echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf
#sudo a2enconf servername
```

- *Restart webserver*
```sh
# service apache2 restart
# service memcached restart
```

**REBOOT tất cả các node**

- *Truy cập vào địa chỉ http://172.16.69.200/horizon*

<img src="http://i.imgur.com/3QkgCiU.png">

*Đăng nhập với username/password: admin/osjuno123a@*

<a name="C"></a>
#C . Kiểm tra lại quá trình cài đặt

<a name="a"></a>
####a. Kiểm tra hoạt động của các dịch vụ
<ul>
Vào controller node kiểm tra các dịch vụ đã hoạt động tốt chưa
</ul>
- *Kiểm tra các dịch vụ của nova* `# nova-manage service list`
<img src="http://i.imgur.com/k1kd1v0.png">
- *Kiểm tra các dịch vụ của neutron* `#neutron agent-list`
<img src="http://i.imgur.com/uirqCpY.png">

- *Truy cập vào địa chỉ http://172.16.69.200/horizon*

<img src="http://i.imgur.com/3QkgCiU.png">
<b name="b"></a>
####b. Tạo external network

- *Chọn admin -> networks*

<img src="http://i.imgur.com/kAVfLp3.png">

- *Chọn Create network*

<img src="http://i.imgur.com/5X7LGiY.png">

- *Nhập các thông tin -> Create network*

<img src="http://i.imgur.com/kbDtE51.png">

- #####*Tạo subnet cho external network*

- *Chọn external*

<img src="http://i.imgur.com/TRequFv.png">

- *Chọn Create Subnet*

<img src="http://i.imgur.com/66rJFKD.png">

- *Nhập thông tin cần thiết -> Next*

<img src="http://i.imgur.com/HQvNhy4.png">

- *Nhập các địa chỉ được cho phép sử dụng ở dải external ->create*

<img src="http://i.imgur.com/raurBKa.png">

- *Thông báo như sau là thành công*

<img src="http://i.imgur.com/8tZjMwS.png">

<a name="c"></a>
####c. Tạo internal network

- *Để tạo mới một internal network: Chọn Project -> Networks*

<img src="http://i.imgur.com/RZkuboT.png">

- *Chọn Create Network*

<img src="http://i.imgur.com/gwaGt4q.png">

- *Nhập các thông tin cho internal network -> Next*

<img src="http://i.imgur.com/yRCKhTG.png">

- *Tạo subnet cho internal network*

<img src="http://i.imgur.com/K3QJNA8.png">

- *Add các địa chỉ được sử dụng trong dải internal -> create*

<img src="http://i.imgur.com/Zxe8xH3.png">

<a name="d"></a>
####d. Tạo router
- *Để tạo router chọn Project -> Network -> Router* 

<img src="http://i.imgur.com/qjBonTJ.png">
- *Chọn create router*

<img src="http://i.imgur.com/AjATj3F.png">

- *Đặt tên cho router -> chọn Next*

<img src="http://i.imgur.com/OU8HvR4.png">

- *Đặt gateway cho router cho phép ra mạng bên ngoài*

<img src="http://i.imgur.com/fkNBtu2.png">

- *Chọn external -> Set Gateway*

<img src="http://i.imgur.com/flaSQzz.png">

<ul>
**Thêm 1 interface mới cho mạng bên trong đi qa router**
</ul>

- *Click chọn Router*

<img src="http://i.imgur.com/XmbJ8UT.png">

- *Chọn Internal network -> Add interface*

<img src="http://i.imgur.com/koBDnAl.png">


<a name="e"></a>
####e. Kiểm tra lại mô hình mạng

- *Để kiểm tra mô hình mạng ta chọn Project -> Network -> Network Topology*

<img src="http://i.imgur.com/hsgIU4q.png">

- *Chọn Normal*

<img src="http://i.imgur.com/JiWtkP8.png">

<a name="f"></a>
####f. Khởi tạo 1 instances

- *Để khởi tạo 1 instace ta chọn Project -> Compute -> Instances*

<img src="http://i.imgur.com/MoSNybV.png">

-  *Chọn Launch instance*

<img src="http://i.imgur.com/uLZxEBQ.png">

- *Chọn các cấu hình như sau -> Launch*

<img src="http://i.imgur.com/EVZFa53.png">

- *Tại phần Network chọn Internal (click vào dấu +) -> Launch*

<img src="http://i.imgur.com/e7sFKkT.png">

- *Clik chọn vào instance vừa khởi tạo* 

<img src="http://i.imgur.com/2tEUhwk.png">

- *Chọn console*

<img src="http://i.imgur.com/IYpGRk9.png">

- *Đăng nhập với username/password hiển thị trên màn hình*

<img src="http://i.imgur.com/n7kUSt3.png">

- *Kiểm tra kết nối của instace*

<img src="http://i.imgur.com/3rULjOD.png">

**Cài đặt openstack thành công**


<a name="D"></a>
#D. Tài liệu tham khảo

http://docs.openstack.org/juno/install-guide/install/apt/content/

https://github.com/vietstacker/openstack-juno-multinode-U14.04-v1

Video hướng dẫn sử dụng dashboard https://www.youtube.com/watch?v=O119UIscdvg

Video hướng dẫn cài đặt Openstack Juno bằng shell script https://www.youtube.com/watch?v=IaZtWQmDjks

