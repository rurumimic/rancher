# Rancher 1.6.14

[Documentation](https://rancher.com/docs/rancher/v1.6/en/)

## CentOS 7

```bash
vagrant up
vagrant ssh rancher
```

(option)

```bash
# vagrant box add generic/centos7 --provider "virtualbox" --insecure
# vagrant plugin uninstall vagrant-vbguest
```

## Docker 17.12.0-ce

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce-17.12.0.ce-1.el7.centos docker-ce-cli-17.12.0.ce-1.el7.centos containerd.io
sudo systemctl start docker
sudo systemctl enable docker
sudo groupadd docker
sudo usermod -aG docker $USER
exit
```

```bash
# vagrant ssh rancher
docker run hello-world
```

## External Database: MariaDB

- MariaDB 10.3.10: [Downloads](https://downloads.mariadb.org/mariadb/10.3.10/)

```bash
vagrant ssh db
```

```bash
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash -s -- --mariadb-server-version="mariadb-10.3.10"
sudo yum install -y MariaDB-server MariaDB-client MariaDB-shared
```

```bash
sudo vi /etc/my.cnf

[mysqld]
  bind-address = 0.0.0.0
```

```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo /usr/bin/mysqladmin -u root password '1234'
sudo netstat -ntlp | grep mysqld
sudo firewall-cmd --add-port=3306/tcp 
sudo firewall-cmd --permanent --add-port=3306/tcp
```

### Create a database and a user

```bash
mysql -u root -p
# 1234
```

```sql
select Host,User,password from mysql.user;
GRANT ALL ON *.* TO 'root'@'%' IDENTIFIED BY '1234';
FLUSH PRIVILEGES;
```

## Rancher 1.6.14

### Start

```bash
docker run -d --restart=unless-stopped -p 8080:8080 --name rancher rancher/server:v1.6.14
```
