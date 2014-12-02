#OpenStack �ֶ���װ�ֲᣨIcehouse��

�����������ͻ�ӭת�������뱣��ԭ������Ϣ!      
���ߣ�[����] �Ƽ��㹤��ʦ�����ݿ���ʵ����    
���ͣ�[http://yongluo2013.github.io/](http://yongluo2013.github.io/)    
΢����[http://weibo.com/u/1704250760/](http://weibo.com/u/1704250760/)  

##����ܹ�

Ϊ�˸��õ�չ��OpenStack������ֲ�ʽ������ص㣬�Լ��߼��������õ����𣬱�ʵ�鲻����All in One �Ĳ���ģʽ�����ǲ��ö�ڵ�ֿ�����ķ�ʽ���������ѧϰ�о���

![architecture](/installation/images/architecture.png)

##��������

![networking](/installation/images/networking.png)

##����׼��

��ʵ�����Virtualbox Windows ����Ϊ���⻯ƽ̨��ģ����Ӧ���������������������������Ҫ������ʵ�����������˲������ֱ���滻Ϊ�����������Ӧ�����ã���ԭ����ͬ��


Virtualbox ���ص�ַ��https://www.virtualbox.org/wiki/Downloads

###��������

��Ҫ�½�3����������Net0��Net1��Net2������virtual box �ж�Ӧ�������¡�

	Net0:
		Network name: VirtualBox  host-only Ethernet Adapter#2
		Purpose: administrator / management network
		IP block: 10.20.0.0/24
		DHCP: disable
		Linux device: eth0

	Net1:
		Network name: VirtualBox  host-only Ethernet Adapter#3
		Purpose: public network
		DHCP: disable
		IP block: 172.16.0.0/24
		Linux device: eth1

	Net2��
		Network name: VirtualBox  host-only Ethernet Adapter#4
		Purpose: Storage/private network
		DHCP: disable
		IP block: 192.168.4.0/24
		Linux device: eth2

###�����

��Ҫ�½�3�������VM0��VM1��VM2�����Ӧ�������¡�

	VM0��
		Name: controller0
		vCPU:1
		Memory :1G
		Disk:30G
		Networks: net1

	VM1��
		Name : network0
		vCPU:1
		Memory :1G
		Disk:30G
		Network:net1,net2,net3

	VM2��
		Name: compute0
		vCPU:2
		Memory :2G
		Disk:30G
		Networks:net1,net3


###��������

	controller0 
	     eth0:10.20.0.10   (management network)
	     eht1:(disabled)
	     eht2:(disabled)

	network0
	     eth0:10.20.0.20    (management network)
	     eht1:172.16.0.20   (public/external network)
	     eht2:192.168.4.20  (private network)

	compute0
	     eth0:10.20.0.30   (management network)
	     eht1:(disabled)
	     eht2:192.168.4.30  (private network)

	compute1  (optional)
	     eth0:10.20.0.31   (management network)
	     eht1:(disabled)
	     eht2:192.168.4.31  (private network)

###����ϵͳ׼��

��ʵ��ʹ��Linux ���а� CentOS 6.5 x86_64���ڰ�װ����ϵͳ�����У�ѡ��ĳ�ʼ��װ��Ϊ����������װ������װ���ϵͳ�Ժ���Ҫ������������YUM �ֿ⡣

ISO�ļ����أ�http://mirrors.163.com/centos/6.5/isos/x86_64/CentOS-6.5-x86_64-bin-DVD1.iso 

EPELԴ: http://dl.fedoraproject.org/pub/epel/6/x86_64/

RDOԴ:  http://repos.fedorapeople.org/repos/openstack/openstack-icehouse/

�Զ�����ִ���������ɣ�Դ��װ��ɺ��������RPM��������������kernel ��Ҫ������������ϵͳ��

	yum install -y http://repos.fedorapeople.org/repos/openstack/openstack-icehouse/rdo-release-icehouse-4.noarch.rpm
	yum install -y http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
	yum update -y
	reboot -h 0

���������Կ�ʼ��װ��������

###�������ã�all nodes��

����������Ҫ��ÿһ���ڵ㶼ִ�С�

�޸�hosts �ļ�

	vi /etc/hosts

	127.0.0.1    localhost
	::1          localhost 
	10.20.0.10   controller0 
	10.20.0.20   network0
	10.20.0.30   compute0


���� selinux 

	vi /etc/selinux/config
	SELINUX=disabled


��װNTP ����

	yum install ntp -y
	service ntpd start
	chkconfig ntpd on

�޸�NTP�����ļ������ô�controller0ʱ��ͬ����(����controller0����)


	vi /etc/ntp.conf

	server 10.20.0.10
	fudge  10.20.0.10 stratum 10  # LCL is unsynchronized


����ͬ�������ʱ��ͬ�������Ƿ���ȷ��(����controller0����)

	ntpdate -u 10.20.0.10
	service ntpd restart
	ntpq -p

��շ���ǽ����

	vi /etc/sysconfig/iptables
	*filter
	:INPUT ACCEPT [0:0]
	:FORWARD ACCEPT [0:0]
	:OUTPUT ACCEPT [0:0]
	COMMIT

��������ǽ���鿴�Ƿ���Ч

	service iptables restart
	iptables -L

��װopenstack-utils,�������ֱ�ӿ���ͨ�������з�ʽ�޸������ļ�


	yum install -y openstack-utils


###��������װ�����ã�controller0 node��

�����������NTP ����MySQL���ݿ�����AMQP���񣬱�ʵ������MySQL ��Qpid ��Ϊ�����������ʵ�֡�


�޸�NTP�����ļ������ô�127.127.1.0 ʱ��ͬ����

	vi /etc/ntp.conf
	server 127.127.1.0

����ntp service

	service ntpd restart

MySQL ����װ

	yum install -y mysql mysql-server MySQL-python

�޸�MySQL����

	vi /etc/my.cnf
	[mysqld]
	bind-address = 0.0.0.0
	default-storage-engine = innodb
	innodb_file_per_table
	collation-server = utf8_general_ci
	init-connect = 'SET NAMES utf8'
	character-set-server = utf8

����MySQL����

	service mysqld start
	chkconfig mysqld on

����ʽ����MySQL root ���룬��������Ϊ��openstack��

	mysql_secure_installation


Qpid ��װ��Ϣ�������ÿͻ��˲���Ҫ��֤ʹ�÷���

	yum install -y qpid-cpp-server

	vi /etc/qpidd.conf
	auth=no


�����޸ĺ�����Qpid��̨����

	service qpidd start
	chkconfig qpidd on


##���ƽڵ㰲װ��controller0��

����������

	vi /etc/sysconfig/network
	HOSTNAME=controller0

��������

	vi /etc/sysconfig/network-scripts/ifcfg-eth0

	DEVICE=eth0
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=10.20.0.10
	NETMASK=255.255.255.0


���������ļ��޸���������������

	serice network restart


###Keyston ��װ������

��װkeystone ��

	yum install openstack-keystone python-keystoneclient -y

Ϊkeystone ����admin �˻��� tokn


	ADMIN_TOKEN=$(openssl rand -hex 10)
	echo $ADMIN_TOKEN
	openstack-config --set /etc/keystone/keystone.conf DEFAULT admin_token $ADMIN_TOKEN


������������


	openstack-config --set /etc/keystone/keystone.conf sql connection mysql://keystone:openstack@controller0/keystone
	openstack-config --set /etc/keystone/keystone.conf DEFAULT debug True
	openstack-config --set /etc/keystone/keystone.conf DEFAULT verbose True

����Keystone �� PKI tokens 


	keystone-manage pki_setup --keystone-user keystone --keystone-group keystone
	chown -R keystone:keystone /etc/keystone/ssl
	chmod -R o-rwx /etc/keystone/ssl


ΪKeystone ����

	mysql -uroot -popenstack -e "CREATE DATABASE keystone;"
	mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'openstack';"
	mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'controller0' IDENTIFIED BY 'openstack';"
	mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'openstack';"

��ʼ��Keystone���ݿ�

	su -s /bin/sh -c "keystone-manage db_sync" 

Ҳ����ֱ����openstack-db ���߳�ʼ���ݿ�

	openstack-db --init --service keystone --password openstack

����keystone ����

	service openstack-keystone start
	chkconfig openstack-keystone on

������֤��Ϣ

	export OS_SERVICE_TOKEN=`echo $ADMIN_TOKEN`
	export OS_SERVICE_ENDPOINT=http://controller0:35357/v2.0


��������Ա��ϵͳ����ʹ�õ��⻧

	keystone tenant-create --name=admin --description="Admin Tenant"
	keystone tenant-create --name=service --description="Service Tenant"


��������Ա�û�

	keystone user-create --name=admin --pass=admin --email=admin@example.com

��������Ա��ɫ


	keystone role-create --name=admin


Ϊ����Ա�û�����"����Ա"��ɫ


	keystone user-role-add --user=admin --tenant=admin --role=admin


Ϊkeystone ������ endpoints


	keystone service-create --name=keystone --type=identity --description="Keystone Identity Service"


Ϊkeystone ���� servie �� endpoint ����


	keystone endpoint-create \
	--service-id=$(keystone service-list | awk '/ identity / {print $2}') \
	--publicurl=http://controller0:5000/v2.0 \
	--internalurl=http://controller0:5000/v2.0 \
	--adminurl=http://controller0:35357/v2.0


��֤keystone ��װ����ȷ��

ȡ����ǰ��Token��������Ȼ������½��û�����֤��

	unset OS_SERVICE_TOKEN OS_SERVICE_ENDPOINT

���������з�ʽ��֤

	keystone --os-username=admin --os-password=admin --os-auth-url=http://controller0:35357/v2.0 token-get
	keystone --os-username=admin --os-password=admin --os-tenant-name=admin --os-auth-url=http://controller0:35357/v2.0 token-get


�ú������û���������֤,������֤��Ϣ

	vi ~/keystonerc

	export OS_USERNAME=admin
	export OS_PASSWORD=admin
	export OS_TENANT_NAME=admin
	export OS_AUTH_URL=http://controller0:35357/v2.0


source ���ļ�ʹ����Ч

	source keystonerc
	keystone token-get


Keystone ��װ������

###Glance ��װ������ 

��װGlance �İ�

	yum install openstack-glance python-glanceclient -y

����Glance �������ݿ�


	openstack-config --set /etc/glance/glance-api.conf DEFAULT sql_connection mysql://glance:openstack@controller0/glance
	openstack-config --set /etc/glance/glance-registry.conf DEFAULT sql_connection mysql://glance:openstack@controller0/glance

��ʼ��Glance���ݿ�

	openstack-db --init --service glance --password openstack

����glance �û�


	keystone user-create --name=glance --pass=glance --email=glance@example.com


������service��ɫ

	keystone user-role-add --user=glance --tenant=service --role=admin

����glance ����

	keystone service-create --name=glance --type=image --description="Glance Image Service"


����keystone ��endpoint 

	keystone endpoint-create \
	--service-id=$(keystone service-list | awk '/ image / {print $2}')  \
	--publicurl=http://controller0:9292 \
	--internalurl=http://controller0:9292 \
	--adminurl=http://controller0:9292


��openstack util �޸�glance api �� register �����ļ�

	openstack-config --set /etc/glance/glance-api.conf DEFAULT debug True
	openstack-config --set /etc/glance/glance-api.conf DEFAULT verbose True
	openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/glance/glance-api.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/glance/glance-api.conf keystone_authtoken admin_user glance
	openstack-config --set /etc/glance/glance-api.conf keystone_authtoken admin_password glance
	openstack-config --set /etc/glance/glance-api.conf paste_deploy flavor keystone

	openstack-config --set /etc/glance/glance-registry.conf DEFAULT debug True
	openstack-config --set /etc/glance/glance-registry.conf DEFAULT verbose True
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken admin_user glance
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken admin_password glance
	openstack-config --set /etc/glance/glance-registry.conf paste_deploy flavor keystone



����glance ��ص���������

	service openstack-glance-api start
	service openstack-glance-registry start
	chkconfig openstack-glance-api on
	chkconfig openstack-glance-registry on


������Cirros������֤glance ��װ�Ƿ�ɹ�

	wget http://cdn.download.cirros-cloud.net/0.3.1/cirros-0.3.1-x86_64-disk.img
	glance image-create --progress --name="CirrOS 0.3.1" --disk-format=qcow2  --container-format=ovf --is-public=true < cirros-0.3.1-x86_64-disk.img



�鿴�ո��ϴ���image 

	glance  image-list

�����ʾ��Ӧ��image ��Ϣ˵����װ�ɹ���


###Nova ��װ������

	yum install -y openstack-nova-api openstack-nova-cert openstack-nova-conductor \
	openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler python-novaclient

��keystone�д���nova��Ӧ���û��ͷ���

	keystone user-create --name=nova --pass=nova --email=nova@example.com
	keystone user-role-add --user=nova --tenant=service --role=admin

keystone ע�����

	keystone service-create --name=nova --type=compute --description="Nova Compute Service"

keystone ע��endpoint

	keystone endpoint-create \
	--service-id=$(keystone service-list | awk '/ compute / {print $2}')  \
	--publicurl=http://controller0:8774/v2/%\(tenant_id\)s \
	--internalurl=http://controller0:8774/v2/%\(tenant_id\)s \
	--adminurl=http://controller0:8774/v2/%\(tenant_id\)s

����nova MySQL ����

	openstack-config --set /etc/nova/nova.conf database connection mysql://nova:openstack@controller0/nova

��ʼ�����ݿ�

	openstack-db --init --service nova --password openstack

����nova.conf

	openstack-config --set /etc/nova/nova.conf DEFAULT debug True
	openstack-config --set /etc/nova/nova.conf DEFAULT verbose True
	openstack-config --set /etc/nova/nova.conf DEFAULT rpc_backend qpid 
	openstack-config --set /etc/nova/nova.conf DEFAULT qpid_hostname controller0

	openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 10.20.0.10
	openstack-config --set /etc/nova/nova.conf DEFAULT vncserver_listen 10.20.0.10
	openstack-config --set /etc/nova/nova.conf DEFAULT vncserver_proxyclient_address 10.20.0.10

	openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
	openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/nova/nova.conf keystone_authtoken admin_user nova
	openstack-config --set /etc/nova/nova.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/nova/nova.conf keystone_authtoken admin_password nova



���api-paste.ini �� Keystone��֤��Ϣ

	openstack-config --set /etc/nova/api-paste.ini filter:authtoken paste.filter_factory keystoneclient.middleware.auth_token:filter_factory
	openstack-config --set /etc/nova/api-paste.ini filter:authtoken auth_host controller0
	openstack-config --set /etc/nova/api-paste.ini filter:authtoken admin_tenant_name service
	openstack-config --set /etc/nova/api-paste.ini filter:authtoken admin_user nova
	openstack-config --set /etc/nova/api-paste.ini filter:authtoken admin_password nova

��������

	service openstack-nova-api start
	service openstack-nova-cert start
	service openstack-nova-consoleauth start
	service openstack-nova-scheduler start
	service openstack-nova-conductor start
	service openstack-nova-novncproxy start

��ӵ�ϵͳ����

	chkconfig openstack-nova-api on
	chkconfig openstack-nova-cert on
	chkconfig openstack-nova-consoleauth on
	chkconfig openstack-nova-scheduler on
	chkconfig openstack-nova-conductor on
	chkconfig openstack-nova-novncproxy on


�������Ƿ�����

	nova-manage service list

	root@controller0 ~]# nova-manage service list
	Binary           Host                                 Zone             Status     State Updated_At
	nova-consoleauth controller0                          internal         enabled    :-)   2013-11-12 11:14:56
	nova-cert        controller0                          internal         enabled    :-)   2013-11-12 11:14:56
	nova-scheduler   controller0                          internal         enabled    :-)   2013-11-12 11:14:56
	nova-conductor   controller0                          internal         enabled    :-)   2013-11-12 11:14:56

������

	[root@controller0 ~]# ps -ef|grep nova
	nova      7240     1  1 23:11 ?        00:00:02 /usr/bin/python /usr/bin/nova-api --logfile /var/log/nova/api.log
	nova      7252     1  1 23:11 ?        00:00:01 /usr/bin/python /usr/bin/nova-cert --logfile /var/log/nova/cert.log
	nova      7264     1  1 23:11 ?        00:00:01 /usr/bin/python /usr/bin/nova-consoleauth --logfile /var/log/nova/consoleauth.log
	nova      7276     1  1 23:11 ?        00:00:01 /usr/bin/python /usr/bin/nova-scheduler --logfile /var/log/nova/scheduler.log
	nova      7288     1  1 23:11 ?        00:00:01 /usr/bin/python /usr/bin/nova-conductor --logfile /var/log/nova/conductor.log
	nova      7300     1  0 23:11 ?        00:00:00 /usr/bin/python /usr/bin/nova-novncproxy --web /usr/share/novnc/
	nova      7336  7240  0 23:11 ?        00:00:00 /usr/bin/python /usr/bin/nova-api --logfile /var/log/nova/api.log
	nova      7351  7240  0 23:11 ?        00:00:00 /usr/bin/python /usr/bin/nova-api --logfile /var/log/nova/api.log
	nova      7352  7240  0 23:11 ?        00:00:00 /usr/bin/python /usr/bin/nova-api --logfile /var/log/nova/api.log


###Neutron server��װ������

��װNeutron server ��ذ�

	yum install -y openstack-neutron openstack-neutron-ml2 python-neutronclient

��keystone�д��� Neutron ��Ӧ���û��ͷ���

	keystone user-create --name neutron --pass neutron --email neutron@example.com

	keystone user-role-add --user neutron --tenant service --role admin

	keystone service-create --name neutron --type network --description "OpenStack Networking"

	keystone endpoint-create \
	--service-id $(keystone service-list | awk '/ network / {print $2}') \
	--publicurl http://controller0:9696 \
	--adminurl http://controller0:9696 \
	--internalurl http://controller0:9696

ΪNeutron ��MySQL�����ݿ�

	mysql -uroot -popenstack -e "CREATE DATABASE neutron;"
	mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'openstack';"
	mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'openstack';"
	mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'controller0' IDENTIFIED BY 'openstack';"

����MySQL

	openstack-config --set /etc/neutron/neutron.conf database connection mysql://neutron:openstack@controller0/neutron

����Neutron Keystone ��֤

	openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_user neutron
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_password neutron

����Neutron qpid

	openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend neutron.openstack.common.rpc.impl_qpid
	openstack-config --set /etc/neutron/neutron.conf DEFAULT qpid_hostname controller0


	openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
	openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
	openstack-config --set /etc/neutron/neutron.conf DEFAULT nova_url http://controller0:8774/v2

	openstack-config --set /etc/neutron/neutron.conf DEFAULT nova_admin_username nova
	openstack-config --set /etc/neutron/neutron.conf DEFAULT nova_admin_tenant_id $(keystone tenant-list | awk '/ service / { print $2 }')
	openstack-config --set /etc/neutron/neutron.conf DEFAULT nova_admin_password nova
	openstack-config --set /etc/neutron/neutron.conf DEFAULT nova_admin_auth_url http://controller0:35357/v2.0

����Neutron ml2 plugin ��openvswitch

	ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini

	openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
	openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins router

	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers gre
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types gre
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_gre tunnel_id_ranges 1:1000
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_security_group True

����nova ʹ��Neutron ��Ϊnetwork ����

	openstack-config --set /etc/nova/nova.conf DEFAULT network_api_class nova.network.neutronv2.api.API
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_url http://controller0:9696
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_auth_strategy keystone
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_tenant_name service
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_username neutron
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_password neutron
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_auth_url http://controller0:35357/v2.0
	openstack-config --set /etc/nova/nova.conf DEFAULT linuxnet_interface_driver nova.network.linux_net.LinuxOVSInterfaceDriver
	openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
	openstack-config --set /etc/nova/nova.conf DEFAULT security_group_api neutron

	openstack-config --set /etc/nova/nova.conf DEFAULT service_neutron_metadata_proxy true
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_metadata_proxy_shared_secret METADATA_SECRET

����nova controller �ϵķ���

	service openstack-nova-api restart
	service openstack-nova-scheduler restart
	service openstack-nova-conductor restart


����Neutron server

	service neutron-server start
	chkconfig neutron-server on
	
##��·�ڵ㰲װ��network0 node��

����������

	vi /etc/sysconfig/network
	HOSTNAME=network0

��������

	vi /etc/sysconfig/network-scripts/ifcfg-eth0
	DEVICE=eth0
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=10.20.0.20
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth1
	DEVICE=eth1
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=172.16.0.20
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth2
	DEVICE=eth2
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=192.168.4.20
	NETMASK=255.255.255.0

���������ļ��޸���������������

	serice network restart

�Ȱ�װNeutron ��صİ�

	yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-openvswitch

����ip forward

	vi /etc/sysctl.conf 
	net.ipv4.ip_forward=1
	net.ipv4.conf.all.rp_filter=0
	net.ipv4.conf.default.rp_filter=0

������Ч

	sysctl -p

����Neutron keysone ��֤

	openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_user neutron
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_password neutron

����qpid 

	openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend neutron.openstack.common.rpc.impl_qpid
	openstack-config --set /etc/neutron/neutron.conf DEFAULT qpid_hostname controller0

����Neutron ʹ��ml + openvswitch +gre

	openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
	openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins router

	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers gre
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types gre
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_gre tunnel_id_ranges 1:1000
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ovs local_ip 192.168.4.20
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ovs tunnel_type gre
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ovs enable_tunneling True
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_security_group True

	ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
	cp /etc/init.d/neutron-openvswitch-agent /etc/init.d/neutronopenvswitch-agent.orig
	sed -i 's,plugins/openvswitch/ovs_neutron_plugin.ini,plugin.ini,g' /etc/init.d/neutron-openvswitch-agent

����l3

	openstack-config --set /etc/neutron/l3_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.OVSInterfaceDriver
	openstack-config --set /etc/neutron/l3_agent.ini DEFAULT use_namespaces True

����dhcp agent

	openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.OVSInterfaceDriver
	openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
	openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT use_namespaces True

����metadata agent

	openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT auth_url http://controller0:5000/v2.0
	openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT auth_region regionOne
	openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT admin_tenant_name service
	openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT admin_user neutron
	openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT admin_password neutron
	openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_ip controller0
	openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret METADATA_SECRET

	service openvswitch start
	chkconfig openvswitch on

	ovs-vsctl add-br br-int
	ovs-vsctl add-br br-ex
	ovs-vsctl add-port br-ex eth1

�޸�eth1��br-ext ��������

	vi /etc/sysconfig/network-scripts/ifcfg-eth1
	DEVICE=eth1
	ONBOOT=yes
	BOOTPROTO=none
	PROMISC=yes 

	vi /etc/sysconfig/network-scripts/ifcfg-br-ex

	DEVICE=br-ex
	TYPE=Bridge
	ONBOOT=no
	BOOTPROTO=none

�����������

	service network restart

Ϊbr-ext ���ip

	ip link set br-ex up
	sudo ip addr add 172.16.0.20/24 dev br-ex

����Neutron ����

	service neutron-openvswitch-agent start
	service neutron-l3-agent start
	service neutron-dhcp-agent start
	service neutron-metadata-agent start

	chkconfig neutron-openvswitch-agent on
	chkconfig neutron-l3-agent on
	chkconfig neutron-dhcp-agent on
	chkconfig neutron-metadata-agent on


## ����ڵ㰲װ����compute0 node��


����������

	vi /etc/sysconfig/network
	HOSTNAME=compute0

��������

	vi /etc/sysconfig/network-scripts/ifcfg-eth0
	DEVICE=eth0
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=10.20.0.30
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth1
	DEVICE=eth1
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=172.16.0.30
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth2
	DEVICE=eth2
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=192.168.4.30
	NETMASK=255.255.255.0

���������ļ��޸���������������

	serice network restart

��װnova ��ذ�

	yum install -y openstack-nova-compute

����nova

	openstack-config --set /etc/nova/nova.conf database connection mysql://nova:openstack@controller0/nova

	openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
	openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/nova/nova.conf keystone_authtoken admin_user nova
	openstack-config --set /etc/nova/nova.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/nova/nova.conf keystone_authtoken admin_password nova

	openstack-config --set /etc/nova/nova.conf DEFAULT rpc_backend qpid
	openstack-config --set /etc/nova/nova.conf DEFAULT qpid_hostname controller0

	openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 10.20.0.30
	openstack-config --set /etc/nova/nova.conf DEFAULT vnc_enabled True
	openstack-config --set /etc/nova/nova.conf DEFAULT vncserver_listen 0.0.0.0
	openstack-config --set /etc/nova/nova.conf DEFAULT vncserver_proxyclient_address 10.20.0.30
	openstack-config --set /etc/nova/nova.conf DEFAULT novncproxy_base_url http://controller0:6080/vnc_auto.html
	openstack-config --set /etc/nova/nova.conf libvirt virt_type qemu

	openstack-config --set /etc/nova/nova.conf DEFAULT glance_host controller0


����compute �ڵ����

	service libvirtd start
	service messagebus start
	service openstack-nova-compute start

	chkconfig libvirtd on
	chkconfig messagebus on
	chkconfig openstack-nova-compute on

��controller �ڵ���compute�����Ƿ�����

	nova-manage service list

�������ڵ����

	[root@controller0 ~]# nova-manage service list
	Binary           Host                                 Zone             Status     State Updated_At
	nova-consoleauth controller0                          internal         enabled    :-)   2014-07-19 09:04:18
	nova-cert        controller0                          internal         enabled    :-)   2014-07-19 09:04:19
	nova-conductor   controller0                          internal         enabled    :-)   2014-07-19 09:04:20
	nova-scheduler   controller0                          internal         enabled    :-)   2014-07-19 09:04:20
	nova-compute     compute0                             nova             enabled    :-)   2014-07-19 09:04:19

��װneutron ml2 ��openvswitch agent 

	yum install openstack-neutron-ml2 openstack-neutron-openvswitch


����Neutron Keystone ��֤

	openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_user neutron
	openstack-config --set /etc/neutron/neutron.conf keystone_authtoken admin_password neutron

����Neutron qpid

	openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend neutron.openstack.common.rpc.impl_qpid
	openstack-config --set /etc/neutron/neutron.conf DEFAULT qpid_hostname controller0

����Neutron ʹ�� ml2 for ovs and gre

	openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
	openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins router

	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers gre
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types gre
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_gre tunnel_id_ranges 1:1000
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ovs local_ip 192.168.4.30
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ovs tunnel_type gre
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ovs enable_tunneling True
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
	openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_security_group True

	ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
	cp /etc/init.d/neutron-openvswitch-agent /etc/init.d/neutronopenvswitch-agent.orig
	sed -i 's,plugins/openvswitch/ovs_neutron_plugin.ini,plugin.ini,g' /etc/init.d/neutron-openvswitch-agent


���� Nova ʹ��Neutron �ṩ�������

	openstack-config --set /etc/nova/nova.conf DEFAULT network_api_class nova.network.neutronv2.api.API
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_url http://controller0:9696
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_auth_strategy keystone
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_tenant_name service
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_username neutron
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_password neutron
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_auth_url http://controller0:35357/v2.0
	openstack-config --set /etc/nova/nova.conf DEFAULT linuxnet_interface_driver nova.network.linux_net.LinuxOVSInterfaceDriver
	openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
	openstack-config --set /etc/nova/nova.conf DEFAULT security_group_api neutron

	openstack-config --set /etc/nova/nova.conf DEFAULT service_neutron_metadata_proxy true
	openstack-config --set /etc/nova/nova.conf DEFAULT neutron_metadata_proxy_shared_secret METADATA_SECRET

	service openvswitch start
	chkconfig openvswitch on
	ovs-vsctl add-br br-int

	service openstack-nova-compute restart
	service neutron-openvswitch-agent start
	chkconfig neutron-openvswitch-agent on

���agent �Ƿ���������

	neutron agent-list

����������ʾ

	[root@controller0 ~]# neutron agent-list
	+--------------------------------------+--------------------+----------+-------+----------------+
	| id                                   | agent_type         | host     | alive | admin_state_up |
	+--------------------------------------+--------------------+----------+-------+----------------+
	| 2c5318db-6bc2-4d09-b728-bbdd677b1e72 | L3 agent           | network0 | :-)   | True           |
	| 4a79ff75-6205-46d0-aec1-37f55a8d87ce | Open vSwitch agent | network0 | :-)   | True           |
	| 5a5bd885-4173-4515-98d1-0edc0fdbf556 | Open vSwitch agent | compute0 | :-)   | True           |
	| 5c9218ce-0ebd-494a-b897-5e2df0763837 | DHCP agent         | network0 | :-)   | True           |
	| 76f2069f-ba84-4c36-bfc0-3c129d49cbb1 | Metadata agent     | network0 | :-)   | True           |
	+--------------------------------------+--------------------+----------+-------+----------------+

##������ʼ����

�����ⲿ����

	neutron net-create ext-net --shared --router:external=True

Ϊ�ⲿ�������subnet

	neutron subnet-create ext-net --name ext-subnet \
	--allocation-pool start=172.16.0.100,end=172.16.0.200 \
	--disable-dhcp --gateway 172.16.0.1 172.16.0.0/24

����ס������

���ȴ���demo�û����⻧�Ѿ������ɫ��ϵ

	keystone user-create --name=demo --pass=demo --email=demo@example.com
	keystone tenant-create --name=demo --description="Demo Tenant"
	keystone user-role-add --user=demo --role=_member_ --tenant=demo

�����⻧����demo-net

	neutron net-create demo-net

Ϊ�⻧�������subnet

	neutron subnet-create demo-net --name demo-subnet --gateway 192.168.1.1 192.168.1.0/24


Ϊ�⻧���紴��·�ɣ������ӵ��ⲿ����

	neutron router-create demo-router

��demo-net ���ӵ�·����

	neutron router-interface-add demo-router $(neutron net-show demo-net|awk '/ subnets / { print $4 }')

����demo-router Ĭ������

	neutron router-gateway-set demo-router ext-net


����һ��instance

nova boot --flavor m1.tiny --image $(nova image-list|awk '/ CirrOS / { print $2 }') --nic net-id=$(neutron net-list|awk '/ demo-net / { print $2 }') --security-group default demo-instance1


##Dashboard ��װ

��װDashboard ��ذ�

	yum install memcached python-memcached mod_wsgi openstack-dashboard

����mencached

	vi /etc/openstack-dashboard/local_settings 

	CACHES = {
	'default': {
	'BACKEND' : 'django.core.cache.backends.memcached.MemcachedCache',
	'LOCATION' : '127.0.0.1:11211'
	}
	}

����Keystone hostname

	vi /etc/openstack-dashboard/local_settings 
	OPENSTACK_HOST = "controller0"

����Dashboard ��ط���

	service httpd start
	service memcached start
	chkconfig httpd on
	chkconfig memcached on

���������֤,�û�����admin ���룺admin

	http://10.20.0.10/dashboard


##Cinder ��װ

###Cinder controller ��װ

����controller0 �ڵ㰲װ cinder api

	yum install openstack-cinder -y

����cinder���ݿ�����

	openstack-config --set /etc/cinder/cinder.conf database connection mysql://cinder:openstack@controller0/cinder


��ʼ�����ݿ�

	mysql -uroot -popenstack -e  "CREATE DATABASE cinder;""
	mysql -uroot -popenstack -e  "GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'openstack';"
	mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'openstack';"
	mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'controller0' IDENTIFIED BY 'openstack';"

	su -s /bin/sh -c "cinder-manage db sync" cinder

������openstack-db ���߳�ʼ�����ݿ�

	openstack-db --init --service cinder --password openstack

��Keystone�д���cinder ϵͳ�û�

	keystone user-create --name=cinder --pass=cinder --email=cinder@example.com
	keystone user-role-add --user=cinder --tenant=service --role=admin

��Keystoneע��һ��cinder �� service

	keystone service-create --name=cinder --type=volume --description="OpenStack Block Storage"

����һ�� cinder �� endpoint

	keystone endpoint-create \
	--service-id=$(keystone service-list | awk '/ volume / {print $2}') \
	--publicurl=http://controller0:8776/v1/%\(tenant_id\)s \
	--internalurl=http://controller0:8776/v1/%\(tenant_id\)s \
	--adminurl=http://controller0:8776/v1/%\(tenant_id\)s

��Keystoneע��һ��cinderv2 �� service

	keystone service-create --name=cinderv2 --type=volumev2 --description="OpenStack Block Storage v2"

����һ�� cinderv2 �� endpoint

	keystone endpoint-create \
	--service-id=$(keystone service-list | awk '/ volumev2 / {print $2}') \
	--publicurl=http://controller0:8776/v2/%\(tenant_id\)s \
	--internalurl=http://controller0:8776/v2/%\(tenant_id\)s \
	--adminurl=http://controller0:8776/v2/%\(tenant_id\)s

����cinder Keystone��֤

	openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken admin_user cinder
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken admin_password cinder

����qpid 

	openstack-config --set /etc/cinder/cinder.conf DEFAULT rpc_backend cinder.openstack.common.rpc.impl_qpid
	openstack-config --set /etc/cinder/cinder.conf DEFAULT qpid_hostname controller0

����cinder controller ��ط���

	service openstack-cinder-api start
	service openstack-cinder-scheduler start
	chkconfig openstack-cinder-api on
	chkconfig openstack-cinder-scheduler on

###Cinder block storage �ڵ㰲װ

ִ������Ĳ���֮ǰ����Ȼ��������Ҫ��װ������������!(����ntp,hosts ��)

��ʼ����֮ǰ��Cinder0 ����һ���´��̣�����block �ķ���

	/dev/sdb

����������

	vi /etc/sysconfig/network
	HOSTNAME=cinder0

��������

	vi /etc/sysconfig/network-scripts/ifcfg-eth0
	DEVICE=eth0
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=10.20.0.40
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth1
	DEVICE=eth1
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=172.16.0.40
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth2
	DEVICE=eth2
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=192.168.4.40
	NETMASK=255.255.255.0

���������ļ��޸���������������

	serice network restart


##��������

![include-cinder](/installation/images/include-cinder.png)

��װCinder ��ذ�

	yum install -y openstack-cinder scsi-target-utils

���� LVM physical and logic ����Ϊcinder ��洢��ʵ��

	pvcreate /dev/sdb
	vgcreate cinder-volumes /dev/sdb

Add a filter entry to the devices section in the /etc/lvm/lvm.conf file to keep LVM from scanning devices used by virtual machines

���һ����������֤ �������ɨ�赽LVM

	vi /etc/lvm/lvm.conf

	devices {
	...
	filter = [ "a/sda1/", "a/sdb/", "r/.*/"]
	...
	}

����Keystone ��֤

	openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_uri http://controller0:5000
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_host controller0
	openstack-config --set /etc/cinder/cinder.conf keystone_authtokenauth_protocol http
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken admin_user cinder
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken admin_tenant_name service
	openstack-config --set /etc/cinder/cinder.conf keystone_authtoken admin_password cinder

����qpid

	openstack-config --set /etc/cinder/cinder.conf DEFAULT rpc_backend cinder.openstack.common.rpc.impl_qpid
	openstack-config --set /etc/cinder/cinder.conf DEFAULT qpid_hostname controller0

�������ݿ�����

	openstack-config --set /etc/cinder/cinder.conf database connection mysql://cinder:openstack@controller0/cinder

����Glance server

	openstack-config --set /etc/cinder/cinder.conf DEFAULT glance_host controller0

����cinder-volume �� my_ip , ���ip�����˴洢����������������

	openstack-config --set /etc/cinder/cinder.conf DEFAULT my_ip 192.168.4.40

����  iSCSI target ������ Block Storage volumes

	vi /etc/tgt/targets.conf
	include /etc/cinder/volumes/*

����cinder-volume ����

	service openstack-cinder-volume start
	service tgtd start
	chkconfig openstack-cinder-volume on
	chkconfig tgtd on

##Swift ��װ

###��װ�洢�ڵ�

��ִ������Ĳ���֮ǰ����Ȼ��������Ҫ��װ������������Ŷ!(����ntp,hosts ��)

�ڿ�ʼ����֮ǰ��ΪSwift0 ����һ���´��̣�����Swift ���ݵĴ洢�����磺

	/dev/sdb

���̴����ú�����OSΪ�´��̷���

	fdisk /dev/sdb
	mkfs.xfs /dev/sdb1
	echo "/dev/sdb1 /srv/node/sdb1 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0" >> /etc/fstab
	mkdir -p /srv/node/sdb1
	mount /srv/node/sdb1
	chown -R swift:swift /srv/node


����������

	vi /etc/sysconfig/network
	HOSTNAME=swift0

��������

	vi /etc/sysconfig/network-scripts/ifcfg-eth0
	DEVICE=eth0
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=10.20.0.50
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth1
	DEVICE=eth1
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=172.16.0.50
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth2
	DEVICE=eth2
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=192.168.4.50
	NETMASK=255.255.255.0

���������ļ��޸���������������

	serice network restart

##��������

����ʡȥCinder �ڵ㲿��

![include-swift](/installation/images/include-swift.png)

��װswift storage �ڵ���صİ�

yum install -y openstack-swift-account openstack-swift-container openstack-swift-object xfsprogs xinetd

����object��container ��account �������ļ�

	openstack-config --set /etc/swift/account-server.conf DEFAULT bind_ip 10.20.0.50
	openstack-config --set /etc/swift/container-server.conf DEFAULT bind_ip 10.20.0.50
	openstack-config --set /etc/swift/object-server.conf DEFAULT bind_ip 10.20.0.50


��rsynd �����ļ�������Ҫͬ�����ļ�Ŀ¼

	vi /etc/rsyncd.conf

	uid = swift
	gid = swift
	log file = /var/log/rsyncd.log
	pid file = /var/run/rsyncd.pid
	address = 192.168.4.50

	[account]
	max connections = 2
	path = /srv/node/
	read only = false
	lock file = /var/lock/account.lock

	[container]
	max connections = 2
	path = /srv/node/
	read only = false
	lock file = /var/lock/container.lock

	[object]
	max connections = 2
	path = /srv/node/
	read only = false
	lock file = /var/lock/object.lock


	vi /etc/xinetd.d/rsync
	disable = no

	service xinetd start
	chkconfig xinetd on

Create the swift recon cache directory and set its permissions:

	mkdir -p /var/swift/recon
	chown -R swift:swift /var/swift/recon

###��װswift-proxy ����

Ϊswift ��Keystone �д���һ���û�

	keystone user-create --name=swift --pass=swift --email=swift@example.com

Ϊswift �û����root �û�

	keystone user-role-add --user=swift --tenant=service --role=admin

Ϊswift ���һ������洢����	

	keystone service-create --name=swift --type=object-store --description="OpenStack Object Storage"

Ϊswift ���endpoint

	keystone endpoint-create \
	--service-id=$(keystone service-list | awk '/ object-store / {print $2}') \
	--publicurl='http://controller0:8080/v1/AUTH_%(tenant_id)s' \
	--internalurl='http://controller0:8080/v1/AUTH_%(tenant_id)s' \
	--adminurl=http://controller0:8080

��װswift-proxy ��������

	yum install -y openstack-swift-proxy memcached python-swiftclient python-keystone-auth-token

��������ļ���������ú���copy ���ļ���ÿ��storage �ڵ�

	openstack-config --set /etc/swift/swift.conf swift-hash swift_hash_path_prefix xrfuniounenqjnw
	openstack-config --set /etc/swift/swift.conf swift-hash swift_hash_path_suffix fLIbertYgibbitZ

	scp /etc/swift/swift.conf root@10.20.0.50:/etc/swift/

�޸�memcached Ĭ�ϼ���ip��ַ

	vi /etc/sysconfig/memcached
	OPTIONS="-l 10.20.0.10"

����mencached

	service memcached restart
	chkconfig memcached on

�޸Ĺ�proxy server����

	vi /etc/swift/proxy-server.conf

	openstack-config --set /etc/swift/proxy-server.conf filter:keystone operator_roles Member,admin,swiftoperator

	openstack-config --set /etc/swift/proxy-server.conf filter:authtoken auth_host controller0
	openstack-config --set /etc/swift/proxy-server.conf filter:authtoken auth_port 35357 
	openstack-config --set /etc/swift/proxy-server.conf filter:authtoken admin_user swift
	openstack-config --set /etc/swift/proxy-server.conf filter:authtoken admin_tenant_name service
	openstack-config --set /etc/swift/proxy-server.conf filter:authtoken admin_password swift
	openstack-config --set /etc/swift/proxy-server.conf filter:authtoken delay_auth_decision true

����ring �ļ�

	cd /etc/swift
	swift-ring-builder account.builder create 18 3 1
	swift-ring-builder container.builder create 18 3 1
	swift-ring-builder object.builder create 18 3 1

	swift-ring-builder account.builder add z1-10.20.0.50:6002R10.20.0.50:6005/sdb1 100
	swift-ring-builder container.builder add z1-10.20.0.50:6001R10.20.0.50:6004/sdb1 100
	swift-ring-builder object.builder add z1-10.20.0.50:6000R10.20.0.50:6003/sdb1 100

	swift-ring-builder account.builder
	swift-ring-builder container.builder
	swift-ring-builder object.builder

	swift-ring-builder account.builder rebalance
	swift-ring-builder container.builder rebalance
	swift-ring-builder object.builder rebalance

����ring �ļ���storage �ڵ�

	scp *ring.gz root@10.20.0.50:/etc/swift/

�޸�proxy server ��storage �ڵ�Swift �����ļ���Ȩ��

	ssh root@10.20.0.50 "chown -R swift:swift /etc/swift"
	chown -R swift:swift /etc/swift

��controller0 ������proxy service

	service openstack-swift-proxy start
	chkconfig openstack-swift-proxy on

��Swift0 �� ����storage ����

	service openstack-swift-object start
	service openstack-swift-object-replicator start
	service openstack-swift-object-updater start 
	service openstack-swift-object-auditor start

	service openstack-swift-container start
	service openstack-swift-container-replicator start
	service openstack-swift-container-updater start
	service openstack-swift-container-auditor start

	service openstack-swift-account start
	service openstack-swift-account-replicator start
	service openstack-swift-account-reaper start
	service openstack-swift-account-auditor start

���ÿ�������

	chkconfig openstack-swift-object on
	chkconfig openstack-swift-object-replicator on
	chkconfig openstack-swift-object-updater on 
	chkconfig openstack-swift-object-auditor on

	chkconfig openstack-swift-container on
	chkconfig openstack-swift-container-replicator on
	chkconfig openstack-swift-container-updater on
	chkconfig openstack-swift-container-auditor on

	chkconfig openstack-swift-account on
	chkconfig openstack-swift-account-replicator on
	chkconfig openstack-swift-account-reaper on
	chkconfig openstack-swift-account-auditor on


��controller �ڵ���֤ Swift ��װ

	swift stat

�ϴ������ļ�����

	swift upload myfiles test.txt
	swift upload myfiles test2.txt

���ظ��ϴ����ļ�

	swift download myfiles


##��չһ���µ�swift storage �ڵ�


�ڿ�ʼ����֮ǰ��ΪSwift0 ����һ���´��̣�����Swift ���ݵĴ洢�����磺

	/dev/sdb

���̴����ú�����OSΪ�´��̷���

	fdisk /dev/sdb
	mkfs.xfs /dev/sdb1
	echo "/dev/sdb1 /srv/node/sdb1 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0" >> /etc/fstab
	mkdir -p /srv/node/sdb1
	mount /srv/node/sdb1
	chown -R swift:swift /srv/node


����������

	vi /etc/sysconfig/network
	HOSTNAME=swift1

��������

	vi /etc/sysconfig/network-scripts/ifcfg-eth0
	DEVICE=eth0
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=10.20.0.51
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth1
	DEVICE=eth1
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=172.16.0.51
	NETMASK=255.255.255.0

	vi /etc/sysconfig/network-scripts/ifcfg-eth2
	DEVICE=eth2
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	IPADDR=192.168.4.51
	NETMASK=255.255.255.0

���������ļ��޸���������������

	serice network restart

yum install -y openstack-swift-account openstack-swift-container openstack-swift-object xfsprogs xinetd

����object��container ��account �������ļ�

	openstack-config --set /etc/swift/account-server.conf DEFAULT bind_ip 10.20.0.51
	openstack-config --set /etc/swift/container-server.conf DEFAULT bind_ip 10.20.0.51
	openstack-config --set /etc/swift/object-server.conf DEFAULT bind_ip 10.20.0.51


��rsynd �����ļ�������Ҫͬ�����ļ�Ŀ¼

	vi /etc/rsyncd.conf

	uid = swift
	gid = swift
	log file = /var/log/rsyncd.log
	pid file = /var/run/rsyncd.pid
	address = 192.168.4.51

	[account]
	max connections = 2
	path = /srv/node/
	read only = false
	lock file = /var/lock/account.lock

	[container]
	max connections = 2
	path = /srv/node/
	read only = false
	lock file = /var/lock/container.lock

	[object]
	max connections = 2
	path = /srv/node/
	read only = false
	lock file = /var/lock/object.lock


	vi /etc/xinetd.d/rsync
	disable = no

	service xinetd start
	chkconfig xinetd on

Create the swift recon cache directory and set its permissions:

	mkdir -p /var/swift/recon
	chown -R swift:swift /var/swift/recon

����ƽ��洢

	swift-ring-builder account.builder add z1-10.20.0.51:6002R10.20.0.51:6005/sdb1 100
	swift-ring-builder container.builder add z1-10.20.0.51:6001R10.20.0.51:6004/sdb1 100
	swift-ring-builder object.builder add z1-10.20.0.51:6000R10.20.0.51:6003/sdb1 100

	swift-ring-builder account.builder
	swift-ring-builder container.builder
	swift-ring-builder object.builder

	swift-ring-builder account.builder rebalance
	swift-ring-builder container.builder rebalance
	swift-ring-builder object.builder rebalance
���ˣ�OpenStack ����������нڵ㰲װ��ϣ�


