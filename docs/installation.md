# Installation instructions

### Install dependendies

~~~
#### python3 ##########################
sudo apt-get install python3-dev

#### mySQL ############################
sudo apt-get install mysql-server

#### python packages ##################
sudo python3 -m pip install simplejson
sudo python3 -m pip install jinja2
sudo python3 -m pip install PyMySQL
sudo python3 -m pip install interruptingcow
sudo python3 -m pip install DBUtils
sudo python3 -m pip install CherryPy
~~~

### Clone this repository

### Setup yourcoin daemon
~~~
# create configuration file  ~/.yourcoin/yourcoin.conf ##

rpcconnect=127.0.0.1
rpcport=<rpcport>
rpcuser=yourcoin
rpcpassword=<password>
txindex=1
~~~

~~~
#### first run yourcoin daemon ###############
./yourcoind -daemon -reindex

#### generally run yourcoin daemon ###########
./yourcoind -daemon
~~~

~~~
#### first run dbload.py #####################
cd ~/yourcoin-cce-4.0/scripts
python3 dbload.py -v -n -l

#### generally run dbload.py without options #
python3 dbload.py -v
~~~
mysql database will filled with the blockchain data.

### Create web folder

~~~
#### create web folder ######################
sudo mkdir /var/www/yourcoin-cce-4.0

#### copy files of /assets to /var/www/yourcoin-cce-4.0 ###
sudo cp -R ~/yourcoin-cce-4.0/assets/* /var/www/yourcoin-cce-4.0
sudo chown -R www-data /var/www/yourcoin-cce-4.0
~~~

### Setup Apache2 Webserver

Basic information about Apache2 webserver ( https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04 )

#### Create SSL certificate
https://hallard.me/enable-ssl-for-apache-server-in-5-minutes/
~~~
#### create SSL directory ##################
sudo mkdir /etc/apache2/ssl

#### create SSL certificate and key ########
sudo openssl req -x509 -nodes -days 1095 -newkey rsa:4096 -out /etc/apache2/ssl/server.crt -keyout /etc/apache2/ssl/server.key

#### enable SSL module #####################
sudo a2enmod ssl

####  disable default apache2 configuration #
sudo a2dissite 000-default.conf  # optional #
sudo a2dissite default-ssl.conf  # optional #

#### restart server ########################
sudo /etc/init.d/apache2 restart
~~~

#### Create and enable server configuration file
~~~
#### configuration file ##################### /etc/apache2/sites-available/yourcoin-cce.conf

<VirtualHost         *:80>
    Redirect / https://<example.com>/
</VirtualHost>

<IfModule mod_ssl.c>
	<VirtualHost *:443>
		ServerAdmin admin@<example.com>

		ProxyPreserveHost On
	    	ProxyErrorOverride On
	    	DocumentRoot /var/www/yourcoin-cce-4.0
	    	ProxyPass /image !
	    	ProxyPass /robots.txt !
	    	ProxyPass /css !
	    	ProxyPass /js !
	    	<Proxy *>
	    		Order allow,deny
	    		Allow from all
	    	</Proxy>
	    	ProxyPass / http://localhost:8222/
	    	ProxyPassReverse / http://localhost:8222/
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined


		SSLEngine on
		SSLCertificateFile    /etc/apache2/ssl/server.crt
		SSLCertificateKeyFile /etc/apache2/ssl/server.key

		<FilesMatch "\.(cgi|shtml|phtml|php)$">
				SSLOptions +StdEnvVars
		</FilesMatch>

		<Directory /usr/lib/cgi-bin>
				SSLOptions +StdEnvVars
				Options +ExecCGI
				AddHandler cgi-script .py
		</Directory>

	</VirtualHost>
</IfModule>
~~~

~~~
#### enable configuration ##################
sudo a2ensite yourcoin-cce.conf

#### reload apache2 server #################
sudo /etc/init.d/apache2 reload
~~~

### Enable executable bit of all sh scripts
~~~
cd ~/yourcoin-cce-4.0
sudo chmod +x start_yourcoind.sh
sudo chmod +x stop_yourcoind.sh
sudo chmod +x start_server.sh
sudo chmod +x stop_server.sh
sudo chmod +x run_dbloader.sh
~~~

### This doc created by TonyFord / FairCoin2
~~~
https://github.com/fairchainsx/faircoin2-cce-4.0
https://github.com/fairchainsx/
~~~
