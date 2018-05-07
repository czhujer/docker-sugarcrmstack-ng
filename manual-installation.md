## install sugarcrmstack manually

### OS
instalace centos7 minimal

### basic OS configuring

vypnutí enforcing v selinuxu
 - vi /etc/sysconfig/selinux
    SELINUX=permissive

vypnutí firewallu
 - systemctl disable firewalld
 - systemctl stop firewalld

switch to old way iptables
 - yum erase firewalld* -y && yum install iptables-services -y

start ssh serveru
 - systemctl start sshd
 - systemctl enable sshd

### install apache + php

instalace apache 2.4
 - yum install httpd -y
 - systemctl status httpd
 - systemctl start httpd
 - systemctl restart httpd

instalace SSL
 - yum install mod_ssl -y

instalace php-fpm

#### add repos
 - rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
 - rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
#### enable repo for remi php 7.1
 - set enabled to 1 in section [remi-php71] in file /etc/yum.repos.d/remi-php71.repo
#### install packages
 - yum install -y php-common php-fpm

instalace php 7.1
 - yum install -y php-cli php-common php-gd php-mbstring php-mcrypt php-mysqlnd php-xml php-bcmath php-pecl-zip
dodat moduly pro PHP
 - yum install php-pecl-apcu php-pecl-redis php-pecl-memcached php-opcache php-imap php-pecl-xdebug

fix php config directory (if exists)
 - cd /etc/opt/remi/php71 && mv php.d php.d.old && ln -s /etc/php.d ./

 nastavení session path (php-fpm)
  - (do not edit /etc/php.ini - it doesn't work)
  - see /etc/php-fpm.d/www.conf
  - mkdir mentioned dir ( /var/lib/php/session )
  - chown apache
  - chmod 775 (necessary?)

konfigurace php (prokopnutí https spojení - vyžaduje SSL)
 - allow override for .htaccess
    https://github.com/SugarFactory/puppet-sugarcrmstack/blob/master/templates/apache-conf-sugarcrm-ssl.conf.erb
 - nějak inteligentně odebrat ruby kód
 - scp .htaccess např. z CBRE (může vyžadovat dva kroky - přes lokál)

instalace mysql
 - yum install -y https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
 - yum install -y mysql-community-server

konfigurace apache
 - vi /etc/httpd/conf.d/
    LoadModule proxy_module modules/mod_proxy.so
    LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
    ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://localhost:9000/var/www/html/$1

nastartovat všechny servízy
 - systemctl start <servíza>
 - httpd
 - php-fpm
 - mysqld
 - systemctl enable <servíza>

### install sugarCRM

stažení instalace sugaru
 - wget ...../download
 - ownCloud - Vývoj > Instalace ...

přejmenování download souboru
 - na cosi.zip

unzip cosi.zip
 - unzip cosi.zip ve /var/www/html
 - případně mv do /var/www/html (nezapomenout zvlášť na mv .htaccess)

doplnit obsah .htaccess z jiného serveru
 - rsync / vi

práva
 - chown -R apache:apache /var/www/html

vytvoření uživatele v mysql + přidat práva pouze pro DB sugarcrm
 - mysql
 - CREATE USER 'sugarcrm'@'localhost' IDENTIFIED BY 'heslo';
 - GRANT ALL PRIVILEGES ON sugarcrm.* TO 'sugarcrm'@'localhost';

dodat phpMyAdmin
 - mkdir -p /usr/share/phpMyAdmin && cd /usr/share/phpMyAdmin
 - git clone --depth 1 https://github.com/phpmyadmin/phpmyadmin.git RELEASE_4_8_0_1


### Installing NGINX [optional step]
================

- edit /etc/httpd/conf/ports.conf to
```
Listen 10080
Listen 10443
```

- edit /etc/httpd/conf.d/25-sugar*.conf and set the ports to 10080 and 10443

- copy /etc/puppet/manifests/puppet-nginx.pp from another hosts directory in Config Management repo

- edit this nginx.pp file:

- uncomment self-signed certificates and comment the SugarFactory ones in puppet nginx file

 (switch comments to this (on two places in the file)

 ```
   ssl_cert    => '/etc/pki/tls/certs/localhost.crt',
   ssl_key     => '/etc/pki/tls/private/localhost.key',
 #  ssl_cert    => '/etc/pki/tls/certs/sugarfactory.cz.public.chain.crt',
 #  ssl_key     => '/etc/pki/tls/private/sugarfactory.cz.key',
 ```

- comment rewrite in first vhost (_)

```
 location_cfg_append => {

     # 'rewrite' => '^ https://crm-xxx.sugarfactory.cz$request_uri? permanent',
```

- uncomment this at the beginning

```
 proxy => 'https://default',
```

 - comment www root and CSR policy (twice in the file)

```
 #www_root    => "/var/www/html",
```
