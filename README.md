# -Alt-linux
Установить сеть на альт линукс
Дэмоэкзамен ход и необходимые действия
Hostnamectl set-hostname ИмяХоста.фамилия.domain (ISP.mich.domain)
Exec bash
Во всех устройствах
------ IPS
apt-get update
apt-get install iptables
mkdir /etc/net/ifaces/enp7s2 				#создаем интерфейс
mkdir /etc/net/ifaces/enp7s3
cat <<EOF> /etc/net/ifaces/enp7s2/options		#настр. Режим.раб
BOOTPROTO=static 				#статический адрес
TYPE=eth						#тип интерфейса
EOF						#выйти из режима редактирования
cat <<EOF> /etc/net/ifaces/enp7s3/options
BOOTPROTO=static
TYPE=eth
EOF
cat <<EOF> /etc/net/ifaces/enp7s2/ipv4address 	#создаем файл с ip
172.16.1.1/28 					#присваиваем адрес
EOF
cat <<EOF> /etc/net/ifaces/enp7s3/ipv4address
172.16.2.1/28
EOF
cat <<EOF> /etc/net/ifaces/enp7s1/resolv.conf
	nameserver 8.8.8.8
EOF

cat <<EOF> /etc/net/ifaces/enp7s2/resolv.conf
nameserver 8.8.8.8
EOF
cat <<EOF> /etc/net/ifaces/enp7s3/resolv.conf
nameserver 8.8.8.8
EOF
Vim /etc/net/sysctl.conf
Форвардинг переводишь с 0 на 1 (редактировать ESC+щ, выйти ESC, сохранить SHIFT + ж, wq, ENTER)
	Sysctl -p
Iptables -t nat -A POSTROUTING -s 172.16.1.0/28 -o enp7s1 -j MASQUERADE
Iptables -t nat -A POSTROUTING -s 172.16.2.0/28 -o enp7s1 -j MASQUERADE
Iptables-save > /etc/sysconfig/iptables
Systemctl enable –now iptables

Iptables -t nat -vnL 	#проверка правил iptables
-------HQ-RTR 
Mkdir /etc/net/ifaces/enp7s2.1
Mkdir /etc/net/ifaces/enp7s2.3
Mkdir /etc/net/ifaces/enp7s2.3
Cat <<EOF> /etc/net/ifaces/enp7s2.1/options #создаем vlan 
BOOTPROT=static
HOST=enp7s2
TYPE=vlan
VID=100
VLAN_REORDER_HDR=0
EOF 
Cat <<EOF> /etc/net/ifaces/enp7s1/ipv4route #задаем шлюз
Default via 172.16.1.1
EOF
cat <<EOF> /etc/net/ifaces/enp7s1/resolv.conf #на всех интерфейсах
nameserver 8.8.8.8
EOF
Mkdir /ect/net/ifaces/gre1 
cat <<EOF> /etc/net/ifaces/gre1/options #создаем gre туннель
	TYPE=iptun #тип интерфейса
	TUNTYPE=gre #тип тунеля
	TUNLOCAL=172.16.1.2 #адрес машины, на которой настраиваешь
	TUNREMOTE=172.16.2.2 #адрес той, куда хочешь отправлять
	TUNOPTIONS=’ttl 64’ 
	EOF
-------BR-RTR 
Тоже самое что было выше (в зависимости от задания)
Cat <<EOF> /etc/net/ifaces/enp7s1/ipv4route
Default via 172.16.2.1
EOF
cat <<EOF> /etc/net/ifaces/gre1/options #создаем gre туннель
	TYPE=iptun #тип интерфейса
	TUNTYPE=gre #тип тунеля
	TUNLOCAL=172.16.2.2 #адрес машины, на которой настраиваешь
	TUNREMOTE=172.16.1.2 #адрес той, куда хочешь отправлять
	TUNOPTIONS=’ttl 64’ 
	EOF
---------HQ-SRV переводим нахуй в самом проксмосе на нужный влан блять
Hardware 🡪 network device 🡪 vlan tag
Настраиваем айпи
Useradd -u 2026 sshuser #создаем пользователя sshuser
Usermod -aG wheel sshuser #заносим в группу пиздатых
Passwd sshuser #задаем пароль
Cat <<EOF> /etc/sudoers.d/sshuser #вход в рут без пароля
	Sshuser ALL=(ALL:ALL) NOPASSWD: ALL
	EOF
Cat <<EOF> /etc/openssh/sshd_config 
	MaxAuthTried 2 #2 попытки ввода пароля
	Banner /etc/openssh/baner #задаем параметр для вывода банера
	EOF
Cat <<EOF> /etc/openssh/baner #создаем банер
	Authorized access only
	EOF


