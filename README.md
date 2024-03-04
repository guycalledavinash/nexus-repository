## Server setup 
I had hard time finding the right commands initially but these worked for me:
```
sudo apt install openjdk-8-jdk
java -version
sudo useradd -d /opt/nexus -s /bin/bash nexus
sudo passwd nexus
ulimit -n 65536
sudo nano /etc/security/limits.d/nexus.conf
nexus - nofile 65536
wget https://download.sonatype.com/nexus/3/nexus-3.41.1-01-unix.tar.gz
tar xzf nexus-3.41.1-01-unix.tar.gz
mv nexus-3.41.1-01 /opt/nexus
mv sonatype-work /opt/
chown -R nexus:nexus /opt/nexus /opt/sonatype-work
sudo nano /opt/nexus/bin/nexus.rc
run_as_user="nexus"
sudo nano /opt/nexus/bin/nexus.vmoptions
```
Change the existing RAM congiguration to this:

-Xms1024m

-Xmx1024m

-XX:MaxDirectMemorySize=1024m
```
sudo nano /etc/systemd/system/nexus.service
```
insert [snippet #1](https://github.com/guycalledavinash/nexus-repository/blob/main/installation-snippet-1)
```
sudo systemctl daemon-reload
sudo systemctl start nexus.service
sudo systemctl enable nexus.service
sudo systemctl status nexus.service
sudo apt install nginx -y
sudo systemctl is-enabled nginx
sudo systemctl status nginx
sudo nano /etc/nginx/sites-available/nexus
```
insert [snippet #2](https://github.com/guycalledavinash/nexus-repository/blob/main/installation-snippet-2)
```
sudo ln -s /etc/nginx/sites-available/nexus /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```
Expose port 8081 in security settings and you're good
