# Installing a Debian server

I'm looking for a stable UNIX like distribution for installing Java 9 or newer and Oracle XE as a virtual machine. I looked at Solaris as a first option, you know, same company, easy integration...

I downloaded and installed Solaris x86 on VMware Fusion. So far so good. Then looked for the latest Java [JDK distribution](http://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html) and big surprise was that Oracle does not have a JDK version for its own x64 version. Big disappointment. Same luck at the [OpenJDK prebuilt packages](http://openjdk.java.net/install/index.html) page.

Since I didn't want to (even try to) compile OpenJDK I decided to stick with the linux option. Now, which Linux? I have used RedHat, CentOS, FreeBSD (I know not Linux, blah, blah) and Ubuntu – being this one my favorite but this time I was looking for something light. CentOS would have been my immediate option, but I stopped using it when they moved away from `initd` for version 7. Long story short, I ended up reading about Debian, and its stable version looked very compelling to me, because it offers a small footprint, the apt package manager, init.d, and since I'm familiar with Ubuntu I'm not expecting any big issues during the regular operation.

I will see how easy or difficult is to make OpenJDK and Oracle XE work on it.

## Debian Installation

* Downloaded the [small installation image](https://www.debian.org/distrib/netinst) for amd64.
* Used the graphical installer, which has the same options as the text one, and does not install any additional package.
* Left the root password empty, so that I'm forced to use sudo.
* For Grub I selected `/dev/sda` – I didn't understand what the manual option meant.
* When selecting packages I chose none – no *desktop environment*, nor *standard system utilities*. Really nothing.

Installation: very easy. Every step has a nice an easy to follow description.

## Additional things to do and install

    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get install openssh-server
    sudo apt-get install net-tools

`net-tools` includes commands like `netstat` and `ifconfig` – the latter command needs `sudo` like in `sudo ifconfig`.

## JDK 10

* Download the required https certificates to the system: `sudo apt-get install ca-certificates` – remember we have a system with minimal stuff in it.
* [Download the JDK](http://jdk.java.net/10/) using `wget`.
* Untar it `tar zxf openjdk-10.0.2_linux-x64_bin.tar.gz`
* `sudo mkdir /opt/jdk`
* `sudo mv jdk-10.0.2`
* `sudo chmod -R root:root /opt/jdk`
* Set environment variables `export JAVA_HOME=/opt/jdk/jdk-10.0.2` and `export PATH=$PATH:$JAVA_HOME/bin`

Done.

## Oracle XE

Again, there is no *.deb* or *.tar.gz* installation file for [Oracle XE](http://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html) but found the following [installation process](https://www.davidpashley.com/articles/oracle-install/)... for version 10. The current one is 11.2. Bummer. I was like, do I really need it?

The answer was No. My final goal is to have a set of microservices running with their own redundant database instances. MySQL can do this work with a smaller footprint. Another option would be to use PostgreSQL, but my operations guy is more familiar with MySQL (and does not want to learn another).


## MySQL Server (the wrong one)

The following shows how the process trying to install MySQL got me into installing MariaDB. Debian promotes the use of open source code, hence this was the default option for MySQL.

* Install MySQL `sudo apt-get install mysql-server`
* And run `sudo mysql_secure_installation`

When trying to login to the database, things didn't work as expected and had to run the following to make it work:

```
sudo service mysql stop
sudo mysqld_safe --skip-grant-tables &
mysql -u root

mysql> use mysql;
mysql> update user set password=PASSWORD("NewPasswd"), plugin='mysql_native_password', host='%' where User='root';
mysql> flush privileges;
mysql> quit

sudo kill -TERM <mysqld_safe process id>
sudo service mysql start
mysql -u root -p
```

More details and my answer can be found at [StackExchange](https://unix.stackexchange.com/questions/327120/after-fresh-install-of-mysql-server-cant-log-in-with-mysql-root-u/463429#463429).

## MySQL Server, this time the right one

* Started by stopping `sudo service mysql stop` and removing MariaDB: `sudo apt-get remove mysql-server`, `sudo apt-get remove mysql-client` and `sudo apt autoremove`

A new hope.

* Install lsb-release with `sudo apt-get install lsb-release` – needed for the next step
* Download the debian repository `wget http://repo.mysql.com/mysql-apt-config_0.8.9-1_all.deb`
* The configuration shows a list of options. I have selected *MySQL Server & Cluster (Currently selected: mysql-5.7)*, *MySQL Tools & Connectors (Currently selected: Enabled)*, and *MySQL Preview Packages (Currently selected: Disabled)*. Select *Ok* on the list to finish.
* Update `sudo apt-get update`
* And install the server `sudo apt install mysql-server`

During the process MySQL complained that the data directory already existed, but I chose to ignore it since it was the one that MariaDB created.

Well, at this point the process had apparently finished ok, but `mysql_secure_installation` didn't exist.

* So I had to rerun `sudo apt install mysql-server`
* And now `sudo mysql_secure_installation` worked.

This process can be seen with more details at [TecAdmin.net](https://tecadmin.net/install-mysql-server-on-debian9-stretch/).

## Nginx

* `sudo apt-get install nginx`
* Create a default web root directory `sudo mkdir -p /srv/www/default`
* Create a dummy index.html document in that directory
* Open the configuration file `sudo vi /etc/nginx/sites-enabled/default` and set the default directory to the one created in the previous step
* Restart nginx `sudo service nginx stop`, `sudo service nginx start`

No issues at all.

## Conclusion

About 4 hours to get this running. But happy at the end. Still thinking whether doing it with Debian was worth it or I should have gone with Ubuntu. Time will tell.

