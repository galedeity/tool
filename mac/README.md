- 安裝homebrew
	- /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
	- sudo vi /etc/paths
		~~~
		/opt/homebrew/bin
		~~~
	- brew -v
	- brew services list
	
- 安裝nodejs
	- brew install node
	- node -v
	- npm -v
	
- 安裝mariadb
    - brew install mariadb
	- brew services start mariadb
	- mysql
		~~~
		ALTER USER 'root'@'localhost' IDENTIFIED BY '1234';
		~~~

- 安裝redis
    - brew install redis
	- brew services start redis
	- redis-cli ping

- 安裝nginx
    - brew install nginx
	- vi /opt/homebrew/etc/nginx/nginx.conf
	    ~~~
		#location / {
		    root html;
			index index.html index.htm index.php;
		#}
		location ~ \.php$ {
			fastcgi_pass   127.0.0.1:9000;
			fastcgi_index  index.php;
			fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
			include        fastcgi_params;
		}
		~~~
	- sudo cp /private/etc/php-fpm.conf.default /private/etc/php-fpm.conf
	- sudo vi /private/etc/php-fpm.conf
		~~~
		pid=/var/run/php-fpm.pid
		error_log=/var/log/php-fpm.log
		~~~
	- sudo cp /private/etc/php-fpm.d/www.conf.default /private/etc/php-fpm.d/www.conf
	- sudo php-fpm
	- brew services start nginx
	- vi /opt/homebrew/var/www/test.php
		~~~
		<?php
			echo phpinfo();
		?>
		~~~
	- http://localhost:8080/test.php
	- https://www.phpmyadmin.net/
	- mv ./Download/phpMyAdmin-5.1.0-all-language /opt/homebrew/var/www/phpmyadmin
	- mkdir /opt/homebrew/var/www/phpmyadmin/tmp
	- chmod 777 /opt/homebrew/var/www/phpmyadmin/tmp
	- cp /opt/homebrew/var/www/phpmyadmin/config.sample.inc.php /opt/homebrew/var/www/phpmyadmin/config.inc.php
	- vi /opt/homebrew/var/www/phpmyadmin/config.inc.php
		~~~
		$cfg['blowfish_secret'] = 'qazwsxedcrfvtgbyhnujmikolp123456';
		$cfg['Servers'][$i]['host'] = '127.0.0.1';
		~~~
	- http://localhost:8080/phpmyadmin

