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

Create separate hosted Maven repositories in Nexus for releases and snapshots. The POM keeps the existing repository paths, `avinash_release` and `avinash_snapshot`, and selects between them based on the project version: versions ending in `-SNAPSHOT` publish to the snapshot repository; other versions publish to the release repository.

### Configure the endpoint

Set `NEXUS_BASE_URL` to the base URL of your own Nexus instance, without a trailing slash. The POM appends `/repository/avinash_release/` or `/repository/avinash_snapshot/`.

PowerShell:

```powershell
$env:NEXUS_BASE_URL = "https://your-nexus-host"
```

Bash:

```sh
export NEXUS_BASE_URL="https://your-nexus-host"
```

Use HTTPS when the Nexus endpoint supports it. Do not send credentials over unencrypted HTTP.

### Configure credentials locally

Copy `settings.xml.example` to your Maven user settings file and replace the placeholders locally with your Nexus username and password. Do not commit the populated file or share its contents.

- Linux/macOS: `~/.m2/settings.xml`
- Windows: `%USERPROFILE%\.m2\settings.xml`

For example, on Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.m2"
Copy-Item .\settings.xml.example "$env:USERPROFILE\.m2\settings.xml"
```

The `<server><id>` value in Maven settings must match the repository `<id>` in `pom.xml`. Both release and snapshot repositories use `nexus`, so Maven uses the matching credentials for either destination. The root-level `settings.xml` is ignored by Git as an additional safeguard.

### Validate and publish

Run local checks first. These do not contact Nexus or publish artifacts:

```sh
mvn clean verify
```

To publish, configure `NEXUS_BASE_URL` and the matching credentials in your Maven user settings, then run:

```sh
mvn deploy
```

Publishing contacts the configured remote Nexus server. Run it only after confirming the endpoint and credentials are correct. Never put real credentials in Git; HTTPS protects them in transit when supported by your Nexus installation.

# Remote Repository

Remote repositories help organizations share libraries and artifacts across projects.
