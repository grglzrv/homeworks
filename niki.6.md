 4. Write a RedHat based configuration file(s) that will create a bonding device from tun0 and tun1.
------------------------------------------------------------------------------------------------------------------

Постановката се разиграва върху CentOS с два NIC-a, като идеята да са в актив-бекъп,
та когато падне единия интерфейс свързаността да продължи от другия.


Описание на бондинг режимите

	mode=0 (Balance-rr) – This mode provides load balancing and fault tolerance.
	mode=1 (active-backup) – This mode provides fault tolerance.
	mode=2 (balance-xor) – This mode provides load balancing and fault tolerance.
	mode=3 (broadcast) – This mode provides fault tolerance.
	mode=4 (802.3ad) – This mode provides load balancing and fault tolerance.
	mode=5 (balance-tlb) – Prerequisite: Ethtool support in the base drivers for retrieving the speed of each slave.
	mode=6 (balance-alb) – Prerequisite: Ethtool support in the base drivers for retrieving the speed of each slave.

Конфигуриране на интерфейсите:
	
	cd /etc/sysconfig/network-scripts/

в директорията създаваме файл с име "ifcfg-bond0" и в него записваме

	DEVICE=bond0
	ONBOOT=yes
	BOOTPROTO=none
	USERCTL=no
	IPADDR=10.10.12.5
	NETMASK=255.255.255.0
	BROADCAST=10.10.12.255
	GATEWAY=10.10.12.1
	BONDING_OPTS="mode=active-backup miimon=100 primary=tun0"

Променяме съдържанието на файловете "ifcfg-tun0" и "ifcfg-tun1",
като им задаваме да са подчинени и кой интерфейс да е главен

	vim ifcfg-tun0

промяна:
	
	DEVICE=tun0
	ONBOOT=yes
	BOOTPROTO=none
	USERCTL=no
	MASTER=bond0
	SLAVE=yes

за tun1

	vim ifcfg-tun1

промяна:

	DEVICE=tun1
	ONBOOT=yes
	BOOTPROTO=none
	USERCTL=no
	MASTER=bond0
	SLAVE=yes


Вече имаме бондинг между tun0 и tun1 в актив-баланс режим.

Трябва и да го дефинираме, това става като създадем файл bonding.conf

	vim /etc/modprobe.d/bonding.conf 

и в него записваме

	alias bond0 bonding

Рестартираме мрежата и бондинга е готов :)

	/etc/init.d/network restart

или

	 service network restart


