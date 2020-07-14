## Build runtime environment

### in ec2 instance

$ sudo yum update
$ sudo yum install -y zip unzip wget vim git yum-utils
$ sudo timedatectl set-timezone Asia/Tokyo
$ sudo setenforce 0
$ sudo vim /etc/selinux/config

```
SELINUX=disabled
```

$ sudo yum install -y httpd
$ sudo vim /etc/httpd/conf.d/msar-production.conf

```
<VirtualHost *:80>
  ServerName 54.199.11.201
  DocumentRoot /home/msar/admin-site/public
  <Directory /home/msar/admin-site/public>
    Options FollowSymLinks
    AllowOverride ALL
    Require all granted
  </Directory>
</VirtualHost>
```

$ sudo systemctl start httpd.service
$ sudo systemctl enable httpd.service
$ sudo yum localinstall -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ sudo yum localinstall -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm
$ sudo yum install -y yum-utils
$ sudo yum-config-manager --enable remi-php74
$ sudo yum update -y
$ sudo yum install -y php php-bcmath php-ctype php-json php-mbstring php-mysql php-tokenizer php-xml php-pdo php-cli php-devel
$ sudo vim /etc/php.ini

```
date.timezone = "Asia/Tokyo"
mbstring.language = Japanese
mbstring.internal_encoding = UTF-8
memory_limit=1024M
upload_max_filesize=512M
post_max_size=1024M
memory_limit=1024M
```

$ sudo php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
$ sudo php -r "unlink('composer-setup.php');"
$ sudo systemctl restart httpd.service

## Deploy application

$ cd /home/
$ sudo mkdir -p msar/admin-site
$ sudo chmod 777 msar/admin-site/
$ git clone https://github.com/uetomae/msar-admin-site.git admin-site
$ cd admin-site/

---

### in local

$ sftp -i ~/.ssh/msar-production.pem centos@54.199.11.201:/home/msar/admin-site

> put _.env
> exit

---

### in ec2 instance

$ mv _.env
$ vim .env

```
:%s/(¥*)//g
```

$ sudo fallocate -l 1G /swapfile
$ sudo dd if=/dev/zero of=/swapfile bs=1024 count=1048576
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
$ composer install
$ sudo php artisan key:generate
$ sudo chmod -R 777 storage/
$ sudo chmod -R 777 bootstrap/cache/
$ sudo systemctl restart httpd
