# sonarqube-installation-ubuntu
sonarqube installation steps ubuntu 


## Install SonarQube on Ubuntu 20.04 LTS

# Introduction
SonarQube is an open-source web-based tool for code quality analysis. SonarQube can analyze a wide range of code in different programming languages through plugins. This guide explains how to install SonarQube on Ubuntu 20.04 LTS.

# Prerequisites
Deploy a fully updated Ubuntu 20.04 LTS server at Vultr with at least 2GB of RAM and 1 vCPU cores.
Create a non-root user with sudo access.
## 1. Install OpenJDK 11 <br>
 <ol>
  
  <li> SSH to your Ubuntu server as a non-root user with sudo access.<br></li>
  <li>Install OpenJDK 11.<br> </li>
     <code>$ sudo apt-get install openjdk-11-jdk -y </code>
  </ol>
  
## 2. Install and Configure PostgreSQL
<ol>
  <br><li>Add the PostgreSQL repository.<br>
<code>$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'</code></li>
  
<br><li>Add the PostgreSQL signing key.<br>
<code>$ wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add - </code></li>

<br><li>Install PostgreSQL.<br>
  <code>$ sudo apt install postgresql postgresql-contrib -y </code></li>

<br><li>Enable the database server to start automatically on reboot.<br>
  <code>$ sudo systemctl enable postgresql </code></li>

<br><li>Start the database server.<br>
  <code>$ sudo systemctl start postgresql </code></li>

<br><li>Change the default PostgreSQL password. <br>
  <code>$ sudo passwd postgres </code></li>



<br><li>Switch to the postgres user. <br>
  <code>$ su - postgres </code></li>

<br><li>Create a user named sonar.<br>
  <code>$ createuser sonar </code></li>

<br><li>Log in to PostgreSQL.<br>
  <code>$ psql </code></li>

<br><li>Set a password for the sonar user. Use a strong password in place of my_strong_password.</br>
<code>ALTER USER sonar WITH ENCRYPTED password 'my_strong_password';</code></li>

<br><li>Create a sonarqube database and set the owner to sonar.<br>
<code>CREATE DATABASE sonarqube OWNER sonar;</code></li>

<br><li>Grant all the privileges on the sonarqube database to the sonar user.<br>
<code>GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar; </code></li>

<br><li>Exit PostgreSQL.<br>
<code>\q </code></li>

<br><li>Return to your non-root sudo user account.<br>
<code>$ exit </code></li>
</ol>

## <br>3. Download and Install SonarQube<br>
<ol>
<br><li>Install the zip utility, which is needed to unzip the SonarQube files.<br>
  <code>$ sudo apt-get install zip -y</code> </li>

  <br><li>Locate the latest download URL from the SonarQube official download page.</li> <br>

 <li>Download the SonarQube distribution files.<br>
<code>$ sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-<VERSION_NUMBER>.zip</code> </li>

<br><li>Unzip the downloaded file.<br>
 <code>sudo unzip sonarqube-<VERSION_NUMBER>.zip</code> </li>

<br><li>Move the unzipped files to /opt/sonarqube directory<br>
<code>sudo mv sonarqube-<VERSION_NUMBER> /opt/sonarqube</code> </li>
  </ol>

## <br>4. Add SonarQube Group and User<br>
Create a dedicated user and group for SonarQube, which can not run as the root user.

<ol>
<br><li>Create a sonar group.<br>
  <code>$ sudo groupadd sonar</code> </li>

<br><li> Create a sonar user and set /opt/sonarqube as the home directory.<br>
  <code>$ sudo useradd -d /opt/sonarqube -g sonar sonar</code> </li>

<br><li>Grant the sonar user access to the /opt/sonarqube directory.<br>
<code>$ sudo chown sonar:sonar /opt/sonarqube -R </code>

</ol>

## <br>5. Configure SonarQube<br>
<ol>
<br><li>Edit the SonarQube configuration file.<br>
<code>$ sudo nano /opt/sonarqube/conf/sonar.properties </code> </li>

<br> <li>  Find the following lines: <br>

  <code>#sonar.jdbc.username=<br></code>
  <br><code>#sonar.jdbc.password=<br></code>  </li>
  
<br><li>Uncomment the lines, and add the database user and password you created in Step 2.<br>
  <code>sonar.jdbc.username=sonar<br></code>
  <br><code>sonar.jdbc.password=my_strong_password</code></li>
  
<br> <li>Below those two lines, add the sonar.jdbc.url.<br>
  <code>sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube </code> </li>

  <br> <li>Save and exit the file.</li><br>

  <br><li>Edit the sonar script file.<br>
  <code>$ sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh</code> </li>

  <br><li>About 50 lines down, locate this line:<br>
  <code>#RUN_AS_USER=</code> </li>

  <br><li>Uncomment the line and change it to:<br>
  <code>RUN_AS_USER=sonar</code> </li>
  <br><li>Save and exit the file.</li> <br>
</ol>

## 6. Setup Systemd service
<ol>
  <br><li>Create a systemd service file to start SonarQube at system boot.<br>
  <code>$ sudo nano /etc/systemd/system/sonar.service </code> </li>

  <br> <li>Paste the following lines to the file.<br>
```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
  
```
  
  <br> <li>Save and exit the file.</li> <br>

<br> <li>Enable the SonarQube service to run at system startup.<br>
<code>$ sudo systemctl enable sonar</code></li>

  <br><li>Start the SonarQube service.<br>
  <code>$ sudo systemctl start sonar </code> </li>

  <br> <li>Check the service status. <br>
  <code>$ sudo systemctl status sonar </code> </li>
  </ol>
  
 ## <br>7. Modify Kernel System Limits<br>
SonarQube uses Elasticsearch to store its indices in an MMap FS directory. It requires some changes to the system defaults.
<ol>
<br><li>Edit the sysctl configuration file.<br>
  <code>$ sudo nano /etc/sysctl.conf</code></li>

  <br> <li>Add the following lines.<br>

```
vm.max_map_count=262144
fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
  
  <br> <li>Save and exit the file.<br></li>

<br> <li>Reboot the system to apply the changes.<br>
  <code>$ sudo reboot</code>
</ol>

## <br>8. Access SonarQube Web Interface<br>
Access SonarQube in a web browser at your server's IP address on port 9000. For example:

<br>http://192.0.2.123:9000 <br>
<br>Log in with username admin and password admin. SonarQube will prompt you to change your password.

  
