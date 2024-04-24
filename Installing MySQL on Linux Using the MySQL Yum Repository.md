# Installing MySQL on Linux Using the MySQL Yum Repository

## Installing MySql server

1. add yum [repository](https://dev.mysql.com/downloads/repo/yum/)

   ```bash
   yum install https://dev.mysql.com/get/mysql80-community-release-el7-11.noarch.rpm
   ```

2. selecting a release series

   ```bash
   yum repolist all | grep mysql
   ```

3. To install the latest release from a specific series other than the latest bugfix series, disable the bug subrepository for the latest bugfix series and enable the subrepository for the specific series before running the installation command. If your platform supports the **yum-config-manager** or **dnf config-manager** command, you can do that by issuing the following commands to disable the subrepository for the 8.0 series and enable the one for the innovation track:

   ```bash
   yum-config-manager --disable mysql80-community
   yum-config-manager --enable mysql-innovation-community
   ```

   ARM Support

   ARM 64-bit (aarch64) is supported on Oracle Linux 7 and requires the Oracle Linux 7 Software Collections Repository (ol7_software_collections). For example, to install the server:

   ```terminal
   yum-config-manager --enable ol7_software_collections
   yum install mysql-community-server
   ```

4. installing mysql

   ```bash
   yum install mysql-community-server
   ```

5. starting the mysql server

   ```bash
   systemctl start mysqld
   ```

6. change the root password 

   ```bash
   grep 'temporary password' /var/log/mysqld.log
   mysql -uroot -p
   mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
   ```

## Changing the default MySQL Data Directory

1. We are going to assume that our new data directory is `/mnt/mysql-data`. It is important to note that this directory should be owned by `mysql:mysql`

   ```bash
   mkdir /mnt/mysql-data
   chown -R mysql:mysql /mnt/mysql-data
   ```

2. #### identify current MySQL data directory

   ```bash
   mysql -u root -p -e "SELECT @@datadir;"
   ```

   after you enter MySQL password, the output should be similar to

   ![Identify MySQL Data Directory](C:\iov\文档\技术\Identify-MySQL-Data-Directory.png)

3. copy MySQL data directory to a new location

   ```bash
   systemctl stop mysqld
   cp -R -p /var/lib/mysql/* /mnt/mysql-data
   ```

4. configure a new MySQL data directory

   ```bash
   vi /etc/my.cnf
   ```

   ```ini
   [mysqld]:
   datadir=/mnt/mysql-data
   socket=/mnt/mysql-data/mysql.sock
   
   [client]:
   port=3306
   socket=/mnt/mysql-data/mysql.sock
   ```

   ![Configure New MySQL Data Directory](C:\iov\文档\技术\Configure-New-MySQL-Data-Directory.png)

5. set SELinux security context to data directory

   This step is only applicable to **RHEL/CentOS** and its derivatives.

   Add the SELinux security context to `/mnt/mysql-data` before restarting MySQL.

   ```bash
   semanage fcontext -a -t mysqld_db_t "/mnt/mysql-data(/.*)?"
   restorecon -R /mnt/mysql-data
   ```

6.  restart the MySQL service

   ```bash
   systemctl start mysqld
   systemctl is-active mysqld
   ```

   use the same command as in **Step 2** to verify the location of the new data directory

   ```bash
   mysql -u root -p -e "SELECT @@datadir;"
   ```

   ![Verify MySQL New Data Directory](C:\iov\文档\技术\Verify-MySQL-New-Data-Directory.png)
