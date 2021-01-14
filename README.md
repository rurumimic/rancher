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
sudo yum update -y
sudo yum install -y yum-utils yum-plugin-versionlock
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce-17.12.0.ce-1.el7.centos containerd.io
sudo yum versionlock docker-ce-17.12.0.ce-1.el7.centos containerd.io
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
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '1234';
FLUSH PRIVILEGES;
```

---

## Rancher 1.6.14

### Start

```bash
docker run -d --restart=unless-stopped -p 8080:8080 --name rancher rancher/server:v1.6.14
```

### SELinux

- [Using Rancher with SELinux enabled - RHEL/CentOS](https://rancher.com/docs/rancher/v1.6/en/installing-rancher/selinux/)

#### Validate

```bash
rpm -q container-selinux

container-selinux-2.119.2-1.911c772.el7_8.noarch
```

#### Install package to build the SELinux module

```bash
sudo yum install -y selinux-policy-devel
```

#### Build the moudle

```bash
vi virtpatch.te
```

```te
policy_module(virtpatch, 1.0)

gen_require(`
  type svirt_lxc_net_t;
')

allow svirt_lxc_net_t self:netlink_xfrm_socket create_netlink_socket_perms;
```

Build

```bash
make -f /usr/share/selinux/devel/Makefile
```

#### Loading the module

```bash
sudo semodule -i virtpatch.pp
```

```bash
sudo semodule -l | grep virtpatch

virtpatch       1.0
```

### Access Control

1. Admin → Access Control
1. Local
1. Setup an Admin user
   - username: `admin`
   - password: `admin`

### Host Registration URL

1. Admin → Host Registration URL
1. Save: `http://<IP>:8080`

### Add Host

1. Infrastructure → Hosts → Add Host
1. Custom
   - Make sure any security groups or firewalls allow traffic:
     - From and To all other hosts on `UDP` ports `500` and `4500` (for IPsec networking)

Run:

```bash
docker run -e CATTLE_AGENT_IP="<IP>" -e CATTLE_HOST_LABELS="name=<name>" --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.9 http://<IP>:8080/v1/scripts/A87EE293D97216668739:1459172500000:2TJly1UqGE2bz3OSE75t4rAF0
```

```bash
INFO: Running Agent Registration Process, CATTLE_URL=http://<IP>:8080/v1
INFO: Attempting to connect to: http://<IP>:8080/v1
INFO: http://<IP>:8080/v1 is accessible
INFO: Inspecting host capabilities
INFO: Boot2Docker: false
INFO: Host writable: true
INFO: Token: xxxxxxxx
INFO: Running registration
INFO: Printing Environment
INFO: ENV: CATTLE_ACCESS_KEY=B0235B2AA8348E8A5AWF
INFO: ENV: CATTLE_AGENT_IP=<IP>
INFO: ENV: CATTLE_HOME=/var/lib/cattle
INFO: ENV: CATTLE_HOST_LABELS=name=accu
INFO: ENV: CATTLE_REGISTRATION_ACCESS_KEY=registrationToken
INFO: ENV: CATTLE_REGISTRATION_SECRET_KEY=xxxxxxx
INFO: ENV: CATTLE_SECRET_KEY=xxxxxxx
INFO: ENV: CATTLE_URL=http://<IP>:8080/v1
INFO: ENV: DETECTED_CATTLE_AGENT_IP=<IP>
INFO: ENV: RANCHER_AGENT_IMAGE=rancher/agent:v1.2.9
INFO: Launched Rancher Agent: c0523ra73575cb23rgdabeefwweg9317f4465663797dacf883b66
```

### Registry

1. Infrastructure → Registries → Add Registry
1. Custom
   - Address: `https://<aws_account_id>.dkr.ecr.<region>.amazonaws.com`
   - Username
   - Password
