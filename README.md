# sysmon_for_linux-securityonion
Configuration files to ingest Sysmon-for-Linux logs into SecurityOnion

# Manager Node Configurations ver 2.3.160 and earlier
## Add the below Elastic Ingest Parsers for Linux Sysmon to the Manager Node
```
git clone https://github.com/bryant-treacle/Sysmon_for_linux
cd Sysmon_for_linux
sudo cp beats.common /opt/so/saltstack/local/salt/elasticsearch/files/ingest/
sudo cp linux_sysmon /opt/so/saltstack/local/salt/elasticsearch/files/ingest/
sudo so-elasticsearch-restart
```
# Installing Sysmon on Linux Server
## Install Dependencies
```
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

```
## Install Packages
```
sudo apt-get update
sudo apt-get install sysinternalsebpf
sudo apt-get install sysmonforlinux
```
### Install Sysmon with configuration file and verify.
```
git clone https://github.com/bryant-treacle/Sysmon_for_linux
cd Sysmon_for_linux
sudo cp sysmon-4-linux-config.xml /bin
sudo sysmon -i /bin/sysmon-4-linux-config.xml -accepteula
sudo systemctl status sysmon
```
# Installing Filebeat on Linux Server
## Manager Node
Prior to installing Filebeat on the Linux server, run the `so-allow` command to allow beats endpoints to connect.

## Linux Host
### Download Filebeat from Security Onion SOC interface and install package.
```
sudo dpkg -i filebeat-oss-*
cp filebeat.yml /etc/filebeat/
```
### Make the following modifications to the filebeat.yml config
```
  filebeat.inputs:
    paths:
     - /var/log/syslog*
  output.logstash
    hosts: ["MANAGER_NODES_IP_HERE:5044"]
  processors:
    - decpde_xml_wineventlog:
        field: message
        target_field: winlog
```
### Start Filebeat
```
sudo systemctl enable filebeat
sudo systemctl start filebeat
sudo systemctl status filebeat
```
## Adding Linux based Detections to Security Onion
### Playbook
To import Linux based sigma rules from SigmaHQ to Playbook, modify the soctopus pillar in the global.sls file.
```
sudo vim /opt/so/saltstack/local/pillar/global.sls
soctopus:
  playbook:
    rulesets:
      - windows
      - linux

sudo so-soctopus-restart
sudo so-playbook-ruleupdate
```

Additional Sigma rules can be found in the Sigma folder.

### Strelka (Yara) Signatures
Sliver implant Yara signatures are located within the Yara folder. If you would like Zeek extract ELF files and pass them to Strelka, you will need to add the following to the /opt/so/saltstack/default/salt/zeek/fileextraction_defaults.yaml file:
```
- application/x-executable: elf
```
To add local YARA rules, create a directory in /opt/so/saltstack/local/salt/strelka/rules, for example localrules. Inside of /opt/so/saltstack/local/salt/strelka/rules/localrules, add your YARA rules.

# Security Onion Dashboards
![Screenshot](/images/Dashboard.png)
![Screenshot](/images/Dashboard2.png)
![Screenshot](/images/Dashboard3.png)
![Screenshot](/images/Dashboard4.png)
