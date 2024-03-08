Predecessor: [Maven](https://github.com/guycalledavinash/maven)

In production, remote repositories are used for sharing libraries, artifacts, etc.

One such tool is Nexus

If the snapshot and release repository details details are configured in `pom.xml` file, the commands ran in local are reflected in nexus repository, not local repo itself

This should be the local directory for maven: `C:\user\.m2\repo\groupID`

## Server setup 
I had hard time finding the right commands initially but these worked for me:
```
sudo apt install openjdk-8-jdk
java -version
sudo useradd -d /opt/nexus -s /bin/bash nexus
sudo passwd nexus
ulimit -n 65536
sudo nano /etc/security/limits.d/nexus.conf
```
paste this in the editor: `nexus - nofile 65536`
```
wget https://download.sonatype.com/nexus/3/nexus-3.41.1-01-unix.tar.gz
tar xzf nexus-3.41.1-01-unix.tar.gz
sudo mv nexus-3.41.1-01 /opt/nexus
sudo mv sonatype-work /opt/
chown -R nexus:nexus /opt/nexus /opt/sonatype-work
sudo nano /opt/nexus/bin/nexus.rc
```
run_as_user="nexus"
```
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
Expose port *8081* in security settings and you're good

![finish](https://github.com/guycalledavinash/nexus-repository/assets/90386560/adb0ba47-73f8-4e80-93f2-34fb512ad656)

## Integrate local Maven with Nexus
First job is to create two repositories in nexus i.e, snapshot and release

1. Go to Settings > repositories > create repo > maven2(hosted)

![set](https://github.com/guycalledavinash/nexus-repository/assets/90386560/e891e95f-e9a9-4100-9049-80d01b8cd5ed)

2. Give a name, select type and allow redeploy - create both repos

![name](https://github.com/guycalledavinash/nexus-repository/assets/90386560/6c60c1d4-761f-42c1-b218-a0b403896842)

3. After creation, copy their links, paste them in this template in URL section.
```
<distributionManagement>  
		<repository>
			<id>nexus</id>
			<name>Avinash Release Nexus Repo</name>
			<url>http://18.191.242.105:8081/repository/avinash_release/</url>
		</repository>
	
		<snapshotRepository>
			<id>nexus</id>
			<name>Avinash Snapshot Nexus Repo</name>
			<url>http://18.191.242.105:8081/repository/avinash_snapshot/</url>
		</snapshotRepository>	
</distributionManagement>
```
4. Paste all this in 'pom.xml' file in the local - [preview](https://github.com/guycalledavinash/nexus-repository/blob/main/pom.xml)

(make sure to update the IP addresses in `pom.xml` as they keep changing everytime we login)

6. Also update 'settings.xml' from `C:maven > conf > settings.xml` like [this](https://github.com/guycalledavinash/nexus-repository/blob/main/settings.xml)

7. Once everything is setup, run this
```
mvn clean deploy
```
![deploy](https://github.com/guycalledavinash/nexus-repository/assets/90386560/3d267239-4586-4a7d-a611-384d59afae06)

This deploys all the artifact in remote repository

#Remote Repository

Remote repos help companies maintain shared libraries with them.

It contributes to efficiency as companies can use the same artifact for multiple projects like a password encryption plugin
