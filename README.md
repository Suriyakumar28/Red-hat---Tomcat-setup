# Red-hat---Tomcat-setup
Setting up guacamole and tomcat in redhat Full setup file

Fedora [172.00.00.153]

Disable SELINUX
sudo grubby --update-kernel ALL --args selinux=0

Please restart the system after completing the necessary tasks.
reboot

Installing the required packages for Tomcat.
dnf install epel-release -y
sudo dnf config-manager --set-enabled crb
sudo dnf install -y unzip curl make cmake wget gcc gcc-c++ zlib-devel openssl-devel cairo-devel libjpeg-turbo-devel \
libpng-devel libtool uuid-devel freerdp-devel pango-devel libssh2-devel libtelnet-devel libvncserver-devel \
libwebsockets-devel pulseaudio-libs-devel libvorbis-devel libwebp-devel vim
dnf install java-11-openjdk-devel -y
useradd -d /usr/share/tomcat -M -r -s /bin/false tomcat
mkdir /usr/share/tomcat
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.97/bin/apache-tomcat-9.0.97.tar.gz
tar xzf apache-tomcat-9.0.97.tar.gz -C /usr/share/tomcat --strip-components=1
chown -R tomcat:tomcat /usr/share/tomcat
cat > /etc/systemd/system/tomcat.service << 'EOL'
[Fedora [172 00 00 153] 132c2319f4868023bf09d72ac8ab5238.html](https://github.com/user-attachments/files/22142685/Fedora.172.00.00.153.132c2319f4868023bf09d72ac8ab5238.html)
[fedora-rocky.zip](https://github.com/user-attachments/files/22142678/fedora-rocky.zip)

[Unit]
Description=Tomcat Server
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment='JAVA_OPTS=-Djava.awt.headless=true'
Environment=CATALINA_HOME=/usr/share/tomcat
Environment=CATALINA_BASE=/usr/share/tomcat
Environment=CATALINA_PID=/usr/share/tomcat/temp/tomcat.pid
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M'
ExecStart=/usr/share/tomcat/bin/catalina.sh start
ExecStop=/usr/share/tomcat/bin/catalina.sh stop

[Install]
WantedBy=multi-user.target
EOL
systemctl daemon-reload
systemctl start tomcat
systemctl status tomcat
Installing Guacamole Server

wget https://downloads.apache.org/guacamole/1.5.5/source/guacamole-server-1.5.5.tar.gz
tar xzf guacamole-server-1.5.5.tar.gz
cd guacamole-server-1.5.5
./configure --with-systemd-dir=/etc/systemd/system/
make
make install
ldconfig
systemctl start guacd
systemctl status guacd
systemctl enable guacd
Installing Guacamole webapp

mkdir /etc/guacamole
wget https://downloads.apache.org/guacamole/1.5.5/binary/guacamole-1.5.5.war -O /etc/guacamole/guacamole.war
ln -s /etc/guacamole/guacamole.war /usr/share/tomcat/webapps/
sudo systemctl enable tomcat.service
sudo systemctl enable guacd.service
systemctl restart tomcat
systemctl restart guacd
mkdir /etc/guacamole/{extensions,lib}
echo "GUACAMOLE_HOME=/etc/guacamole" >> /etc/default/tomcat
nano /etc/guacamole/guacamole.properties
guacd-hostname: localhost
guacd-port:     4822
user-mapping:   /etc/guacamole/user-mapping.xml
auth-provider:  net.sourceforge.guacamole.net.basic.BasicFileAuthenticationProvider
ln -s /etc/guacamole /usr/share/tomcat/.guacamole
systemctl restart tomcat guacd
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --add-port=4822/tcp --permanent
firewall-cmd --reload
Configure ports

sudo nano /etc/guacamole/guacd.conf
[server]
bind_host = 127.0.0.1
bind_port = 4822
If RDP not working use following code

useradd -M -d /var/lib/guacd/ -r -s /sbin/nologin -c "Guacd User" guacd
mkdir /var/lib/guacd
chown -R guacd: /var/lib/guacd
sed -i 's/daemon/guacd/' /etc/systemd/system/guacd.service
systemctl daemon-reload
systemctl restart guacd
sudo systemctl restart tomcat
Installing DB

sudo dnf install mariadb-server mariadb -y
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo mysql_secure_installation
Using root user to create DB

sudo mysql -u root -p
CREATE DATABASE guacamole_db;
CREATE USER 'guacamole_user'@'localhost' IDENTIFIED BY 'Secret55!';
GRANT SELECT,INSERT,UPDATE,DELETE ON guacamole_db.* TO 'guacamole_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
Configuring DB to connect

cd /tmp
wget https://downloads.apache.org/guacamole/1.5.5/binary/guacamole-auth-jdbc-1.5.5.tar.gz
tar -xzf guacamole-auth-jdbc-1.5.5.tar.gz
cd guacamole-auth-jdbc-1.5.5/mysql/schema
cat *.sql | mysql -u root -p guacamole_db
sudo mkdir -p /etc/guacamole/{extensions,lib}
sudo cp /tmp/guacamole-auth-jdbc-1.5.5/mysql/guacamole-auth-jdbc-mysql-1.5.5.jar /etc/guacamole/extensions/
Adding connector

cd /tmp
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-8.0.33.tar.gz
tar -xzf mysql-connector-j-8.0.33.tar.gz
sudo cp mysql-connector-j-8.0.33/mysql-connector-j-8.0.33.jar /etc/guacamole/lib/
sudo chown -R tomcat:tomcat /etc/guacamole
sudo systemctl restart tomcat guacd
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
LDAP
sudo wget https://downloads.apache.org/guacamole/1.5.5/binary/guacamole-auth-ldap-1.5.5.tar.gz
sudo tar -xzf guacamole-auth-ldap-1.5.5.tar.gz
sudo cp guacamole-auth-ldap-1.5.5/guacamole-auth-ldap-1.5.5.jar /etc/guacamole/extensions/
sudo nano /etc/guacamole/guacamole.properties
# Guacamole Server Configuration
guacd-hostname: localhost
guacd-port: 4822

# MySQL properties
mysql-hostname: localhost
mysql-port: 3306
mysql-database: guacamole_db
mysql-username: guacamole_user
mysql-password: Secret55!

# JDBC Config
mysql-default-max-connections-per-user: 0
mysql-default-max-group-connections-per-user: 0
# LDAP properties
ldap-hostname: 172.00.00.151
ldap-port: 389
ldap-encryption-method: none
ldap-user-base-dn: DC=linuxtest,DC=ca
ldap-username-attribute: sAMAccountName
ldap-search-bind-dn: CN=Administrator,CN=Users,DC=linuxtest,DC=ca
ldap-search-bind-password: Secret55!
ldap-user-search-filter: (objectClass=user)

auth-provider: net.sourceforge.guacamole.net.auth.ldap.LDAPAuthenticationProvider
sudo systemctl restart tomcat guacd
nano /etc/guacamole/user-mapping.xml
<user-mapping>
    <authorize username="username" password="Secret55!">

        <!-- First authorized Remote connection -->
        <connection name="Windows Server RDP">
            <protocol>rdp</protocol>
            <param name="hostname">172.00.00.151</param>
            <param name="port">3389</param>
            <param name="username">administrator</param>
            <param name="password">Secret55!</param>
            <param name="ignore-cert">true</param>
        </connection>

        <!-- Second authorized remote connection -->
        <connection name="Windows 10 RDP">
            <protocol>rdp</protocol>
            <param name="hostname">172.00.00.150</param>
            <param name="port">3389</param>
            <param name="username">Win-Desktop</param>
            <param name="password">Secret55!</param>
            <param name="ignore-cert">true</param>
        </connection>
    </authorize>
</user-mapping>
