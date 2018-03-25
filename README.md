# Siren Solutions Investigate aka Kibi
Instructions for setting up elasticsearch and siren investigate. 

[Siren Website](https://www.siren.io)
> Note you may need to create a login to access the support pages and documentation below.

There are two versions available. I installed the Siren Platform 10 Beta 3 running an internal corporate Linux CentOS server.:octocat:

Stable: [Siren Platform 5.x](https://support.siren.io/support/solutions/articles/17000063389-platform-5-4-3-4)
Requires data to be held in an Elasticsearch cluster

Beta: [Siren Platform 10-beta-3](https://support.siren.io/support/solutions/articles/17000068677-platform-10-0-0-beta-3)

 * Allows the analysis of data in other backends thanks to JDBC connectivity
 * Also includes fully distributed joins for data in Siren/Elasticsearch indexes
 * You can map JDBC datasources as "Virtual Indexes" and navigate between them and Elasticsearch Indexes
 * An improved relational model - allowing relationships between entities to be build and analyzed
 * We will install the necessary jar files on our VM server for the other external datasource connectors.

For more information, see [here.](https://siren.io/siren-10-beta-1-available-multiple-back-ends-distributed-joins-new-datamodel/)

## Installation on your local machine
The quickest way to do an evaluation is on your local virtual machine running Linux or your Mac. Once you have unzipped the file you follow the instructions provided that go like this:

```
unzip siren-platform-10.0.0-beta-3-${OS}
# new terminal
cd siren-platform-10.0.0-beta-3-${OS}/elasticsearch
./bin/elasticsearch
# new terminal
cd siren-platform-10.0.0-beta-3-${OS}/investigate
./bin/investigate
```
> Note - both elasticsearch and investigate run in the foreground so when you do Ctl-C or exit your terminal, the process will terminate.

You access the application with `localhost:5606`

See the Siren documentation for login and passwords.

## Installation on remote host

Download the zip file and expand the files. If you choose the with one with demonstration data, the file is 1.2GB. 

> Note - you will need to open port 5606 on your remote server in addition to port 22 for ssh

You must edit the <your-folder>/siren-investigate/config/investigate.yml file. Change the following entry to:

`server.host: "0.0.0.0"`

### Run elasticsearch and siren investigate in the background

In order to run these two applications in the background we must use a service within the RHEL Linux systemd framework. I created two services that are designed to run both the elasticsearch cluster and siren investigate using the control of `systemctl`. 
There were different types of errors when figuring out this solutions. First, you need to specify a user for the service file. You will get an error because it cannot run as root. Second, you may see syntax errors either at line breaks or equal signs. This is because some versions of CentOS do not like extra spaces. Always expand the parts of the output hidden by ellipses using `sudo systemctl status kibi-es.service -l` Third, bin/bash scripts require you place `bin/bash` in the `ExecStart` section. For regular shell scripts, you can omit the command shell prefix. Fourth, the kibi script required a working directory since the NodeJS modules and other scripts were dependent upon knowing their locations. Once those were fixed, these services ran as expected.

First check your system:
```
systemctl --type=service
cd /etc/systemd/system
ls -l
ls multi-user.target.wants/
```
Then create the following two service files. The syntax is critical. For some versions of CentOS, no spaces are used to separate the sections or between equal signs. Your system may not use `network.target`, instead we use `basic.target`. Many examples will show you network.target `After` service. You can find out by running:
```
sudo systemd-analyze critical-chain
```

Create this file as `sudo nano kibi-es.service`
```
[Unit]
Description=Starting Kibi-elasticsearch
After=basic.target
[Service]
Type=simple
User=<valid userid with permissions>
ExecStart=/bin/bash /folder/siren/sirenprogram-unzipped/elasticsearch/bin/elasticsearch
RestartSec=35
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
The second file is created as `sudo nano kibi-investigate.service`. Notice this service depends on elasticsearch running
```
[Unit]
Description=Starting up Kibi Investigate
After=basic.target kibi-es.service
[Service]
Type=simple
User=<valid userid with permissions>
WorkingDirectory=/folder/siren/sirenprogram-unzipped/siren-investigate
ExecStart=/folder/siren/sirenprogram-unzipped/siren-investigate/bin/investigate
RestartSec=35
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
Start and check everything:
```
sudo systemctl daemon-reload
sudo systemctl start kibi-es.service
sudo systemctl status kibi-es.service
sudo systemctl daemon-reload
sudo systemctl start kibi-investigate.service
sudo systemctl status kibi-investigate.service
```
Then enable these services to restart automatically in case the server is rebooted.
```
sudo systemctl enable kibi-es.service
sudo systemctl enable kibi-investigate.service

```

Access the application on remote host:
`https://<hostname-url>:5606`

Good luck!  :muscle: :octocat:
