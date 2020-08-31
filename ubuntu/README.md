- 用戶名登入root
	- sudo -i
	
- 更新linux版本及軟體
	- apt update && apt upgrade && apt dist-upgrade
	
- 設定台灣時區
	- cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime
	
- 設定開機啟動
	- vi /lib/systemd/system/rc-local.service
		~~~
		[Install]
		WantedBy=multi-user.target
		Alias=rc-local.service
		~~~
	- ln -fs /lib/systemd/system/rc-local.service /etc/systemd/system/rc-local.service
	- touch /etc/rc.local
	- chmod 755 /etc/rc.local
	- vi /etc/rc.local
		~~~
		#!/bin/bash
		~~~
		
- 設定網路
	- ifconfig 查ip及網卡名稱
	- cat /run/systemd/resolve/resolv.conf 查DNS
	- route -n 查路由器
	- dhclient eno1 自動取得ip
	- vi /etc/netplan/50-cloud-init.yaml
		- 手動設定ip
			~~~
			eno1: #外網
				addresses: [x.x.x.x/27]
				gateway4: x.x.x.x
				nameservers:
				addresses: [x.x.x.x]
				dhcp4: false
				dhcp6: false
				optional: true
			eno2: #內網
				addresses: [x.x.x.x/24]
				dhcp4: false
				dhcp6: false
				optional: true
			~~~
		- 自動取得ip
			~~~
			eno1: #外網
				addresses: []
				dhcp4: true
				dhcp6: false
				optional: true
			eno2: #內網
				addresses: [x.x.x.x/24]
				dhcp4: false
				dhcp6: false
				optional: true
			~~~
	- netplan apply 將設定值生效
	- vi /etc/resolv.conf 手動設定DNS
		~~~
		nameserver x.x.x.x 
		~~~
		
- 網路共享
	- vi /etc/sysctl.conf
		~~~
		net.ipv4.ip_forward = 1
		~~~
	- vi /etc/rc.local
		~~~
		iptables -F
		iptables -P INPUT ACCEPT
		iptables -P FORWARD ACCEPT          
		iptables -t nat -A POSTROUTING -o eno1 -j MASQUERADE
		~~~
		
- 架設虛擬主機
	- apt install build-essential dkms unzip wget
	- reboot
	- vi /etc/apt/sources.list
		~~~
		deb http://download.virtualbox.org/virtualbox/debian bionic contrib
		~~~
	- wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | apt-key add -
	- wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | apt-key add -
	- add-apt-repository universe
	- add-apt-repository multiverse
	- apt update && apt upgrade
	- dpkg -r libqt5core5a libqt5widgets5
	- apt install libpng16-16 libqt5core5a libqt5printsupport5 libqt5widgets5 libqt5x11extras5
	- apt -f install
	- apt install virtualbox-5.2
	- wget https://download.virtualbox.org/virtualbox/5.2.30/Oracle_VM_VirtualBox_Extension_Pack-5.2.30.vbox-extpack
		- https://www.virtualbox.org/wiki/Download_Old_Builds_5_2 看版本 ex: 5.2.30
	- VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-5.2.30.vbox-extpack
	- apt install apache2 php php-mysql libapache2-mod-php php-soap php-xml
	- wget https://github.com/phpvirtualbox/phpvirtualbox/archive/5.2-0.zip
		- https://github.com/phpvirtualbox/phpvirtualbox/releases 看版本 ex: 5.2-0
	- unzip 5.2-0.zip
	- mv phpvirtualbox-5.2-0/ /var/www/html/phpvirtualbox
	- chmod 777 /var/www/html/phpvirtualbox/
	- cp /var/www/html/phpvirtualbox/config.php-example /var/www/html/phpvirtualbox/config.php
	- vi /var/www/html/phpvirtualbox/config.php
		~~~
		var $username = 'ubuntu用戶名';
		var $password = 'ubuntu用戶名密碼';
		var $language = 'zh_tw';
		~~~
	- vi /etc/default/virtualbox
		~~~
		VBOXWEB_USER = ubuntu用戶名
		~~~
	- systemctl restart vboxweb-service
	- systemctl restart vboxdrv
	- systemctl restart apache2
	- ufw app list
	- ufw app info "Apache Full"
	- ufw allow in "Apache Full"        
	- ufw app info "Apache"
	- http://x.x.x.x/phpvirtualbox	帳密都是admin
	- 鎖ip設定
		- vi /etc/apache2/apache2.conf
			~~~
			<Directory /var/www/html/phpvirtualbox/>
				Order Deny,Allow
				Deny from all
				Allow from 127.0.0.1
				Allow from x.x.x.x
			</Directory>
			~~~
		- service apache2 restart
	- VBoxManage modifyvm "虛擬主機名稱" --nataliasmode1 proxyonly 設定來源ip不被虛擬主機轉換
	- vi /etc/rc.local
		~~~
		su ubuntu用戶名 -c "VBoxManage startvm '虛擬主機名稱' --type headless"
		~~~
		
- 擴充硬碟容量
	- VBoxManage modifyhd "路徑/虛擬主機名稱.vdi"  --resize 總容量
	- sudo parted
		~~~
		(parted) print
		Fix/Igrone? f
		(parted) q
		~~~
	- sudo gdisk /dev/sda
		~~~
		command (m for help): d
		command (m for help): n
		command (m for help): p
		command (m for help): w
		~~~
	- sudo reboot 
	- sudo resize2fs /dev/sda2
	- df -h

- 架設叢集式資料庫mysql-cluster
	- 執行主機：mgm、db1、db2
		- sudo wget http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.6/mysql-cluster-gpl-7.6.12-linux-glibc2.12-x86_64.tar.gz
		- sudo tar -zxvf mysql-cluster-gpl-7.6.12-linux-glibc2.12-x86_64.tar.gz
		- sudo mv mysql-cluster-gpl-7.6.12-linux-glibc2.12-x86_64 mysql
		- sudo mv mysql /usr/local
	- 執行主機：mgm
		- sudo mkdir /usr/local/mysql/mysql-cluster
		- sudo vi /usr/local/mysql/mysql-cluster/config.ini
			~~~
			[ndbd default]
			NoOfReplicas=2
			DataMemory=256M
			IndexMemory=256M

			[ndb_mgmd]
			HostName=10.0.2.10
			DataDir=/usr/local/mysql/mysql-cluster

			[ndbd]
			HostName=10.0.2.20
			DataDir=/usr/local/mysql/data

			[ndbd]
			HostName=10.0.2.21
			DataDir=/usr/local/mysql/data

			[mysqld]
			HostName=10.0.2.10
			~~~
		- sudo /usr/local/mysql/bin/ndb_mgmd -f /usr/local/mysql/mysql-cluster/config.ini --initial
	- 執行主機：db1、db2
		- sudo vi /etc/my.cnf
			~~~
			[mysqld]
			ndbcluster
			ndb-connectstring=10.0.2.10

			[mysql_cluster]
			ndb-connectstring=10.0.2.10
			~~~
		- sudo /usr/local/mysql/bin/ndbd --initial
	- 執行主機：mgm或db1、db2
		- sudo apt-get install libaio1
		- sudo groupadd mysql
		- sudo useradd mysql -s /sbin/nologin -g mysql
		- sudo chown -R root:root /usr/local/mysql
		- sudo /usr/local/mysql/bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
		- sudo chown -R mysql:mysql /usr/local/mysql/data
		- sudo /usr/local/mysql/support-files/mysql.server start
	- 執行主機：mgm
		- sudo /usr/local/mysql/bin/ndb_mgm -e show
		
- 安裝docker
	- sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
	- curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	- sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"  
	- sudo apt update  
	- sudo apt-get install docker-ce
	- sudo service docker status 查看docker狀態
	- sudo docker version 查看版本
	- 製作images
		~~~
		FROM node:8.16.0
		COPY xxx.js package*.json /app/
		WORKDIR /app
		RUN npm install --save ws && npm install && npm cache clean --force
		CMD node xxx.js
		~~~
	- sudo docker build -t 檔名 .
		~~~
		example:
		sudo docker run 檔名
		-d 背景執行
		-v /aaa:/bbb 容器bbb目錄映射到本機aaa目錄
		--net=host 容器網段與本機1:1對應
		-p 1234:5678 容器port5678映射到本機port1234
		~~~
	- sudo vi /etc/rc.local
		~~~
		docker restart $(docker ps -aq)
		~~~
		
- 安裝Nodejs
	- curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh
	- sudo bash nodesource_setup.sh
	- sudo apt install nodejs
	- sudo apt install build-essential
	- curl -sL https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh -o install_nvm.sh
	- bash install_nvm.sh
	- source ~/.profile
	- nvm ls-remote Latest LTS: Carbon版本
	- nvm install 8.16.0
	- nvm use 8.16.0
	- nvm alias default 8.16.0
	- sudo npm install forever -g
		~~~
		example:
		node 檔名.js
		npm install 封包名 //有缺再裝
		~~~
	- node -v 查看版本
	- sudo vi /etc/rc.local
		~~~
		su ubuntu用戶名 -c "forever start -l 檔名.log --minUptime 1000 --spinSleepTime 1000 --pidFile 檔名.pid -a /home/ubuntu用戶名/檔名.js" 
		~~~
		
- 安裝C
	- sudo apt install gcc
	- sudo apt install libmysqlclient-dev 資料庫封包
		~~~
		example:
		gcc -o 檔名 檔名.c -Wall `mysql_config --cflags --libs`
		~~~
		
- 安裝Golang
	- sudo add-apt-repository ppa:longsleep/golang-backports
	- sudo apt-get update
	- sudo apt-get install golang-go
		~~~
		example:
		go build 檔名.go
		go run 檔名.go
		go get 封包名 //有缺再裝
		~~~
		
- 雲端備份google drive
	- sudo wget -O /usr/bin/gdrive "https://docs.google.com/uc?id=0B3X9GlR6EmbnQ0FtZmJJUXEyRTA&export=download"
	- sudo chmod 777 /usr/bin/gdrive
	- gdrive about //等待輸入授權碼
	- http://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=367116221053-7n0vf5akeru7on6o2fjinrecpdoe99eg.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&state=state 授權成功會出現.gdrive目錄並會產生token文件
	- vi backup.sh
		~~~
		#!/bin/sh
		mkdir backup
		filename=$(date +%Y%m%d%H%M).sql
		mysqldump -u root -p12345678 --databases 資料庫名稱 > backup/$filename
		directoryname=backup$(date +%Y%m%d%H)
		selectdirectory=$(gdrive list --no-header -q "name='$directoryname'")
		[ -z "$selectdirectory" ]                                                             
		case $? in
		1)
			directoryid=$(echo $selectdirectory | cut -d" " -f 1)                             
			;;
		*)
			directoryid=$(echo $(gdrive mkdir $directoryname) | cut -d" " -f 2)               
			;;
		esac
		gdrive upload -p $directoryid backup/$filename                                        
		rm -r backup                                                                          
		directoryname=backup$(date +%Y%m%d%H -d '2 hour ago')                                 
		selectdirectory=$(gdrive list --no-header -q "name='$directoryname'")                  
		[ -z "$selectdirectory" ]
		case $? in
		1)
			gdrive delete -r $(echo $selectdirectory | cut -d" " -f 1)
			;;
		*)
			;;
		esac
		~~~
	- crontab -e
		~~~
		*/5 * * * * /home/ubuntu用戶名/backup.sh
		~~~
	- 還原備份檔
		- gdrive list 查雲端硬碟檔案
		- gdrive download hdjsfjadfhadshfl 假設要下載xxx.sql id為hdjsfjadfhadshfl
        - mysql -u root -p12345678 < xxx.sql